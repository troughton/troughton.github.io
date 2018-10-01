---
layout: post
title:  "Spherical Gaussian Encoding"
date:   2018-09-21 15:37:26 +1200
categories: rendering irradiance-caching spherical-gaussians
---

Spherical Gaussians are a useful tool for encoding precomputed radiance within a scene. Matt Pettineo has [an excellent series](https://mynameismjp.wordpress.com/2016/10/09/sg-series-part-1-a-brief-and-incomplete-history-of-baked-lighting-representations/) describing the technical details and history behind them which I'd suggest reading before the rest of this post.

Recently, I've had need to build spherical Gaussian representations of a scene on the fly in the GPU path tracer I'm building for my Master's project. Unlike, say, spherical harmonics, spherical Gaussian lobes don't form a set of orthonormal bases; they need to be computed with a least-squares solve, which generally requires having access to all radiance samples at once. On the GPU, in a memory-constrained environment, this is highly impractical. 

Instead, games like The Order: 1886 just projected the samples onto the spherical Gaussian lobes [as if the lobes form an orthonormal basis](https://mynameismjp.wordpress.com/2016/10/09/sg-series-part-5-approximating-radiance-and-irradiance-with-sgs/), which gave low-quality results that lack contrast. For my research, I wanted to see if I could do better.

One option, as [Peter-Pike Sloan points out on Twitter](https://twitter.com/PeterPikeSloan/status/1044482721223856128), is to accumulate the raw moments and then multiply by a lobeCount × lobeCount matrix to reconstruct the result. 

After a bunch of experimentation, I found a new equivalent algorithm that enables us to compute the lobe amplitudes in place, without needing a large matrix multiply at the end. The results are as good as a standard least-squares solve if the sample directions are randomly distributed, but deteriorates if the sample directions are correlated (as they would be, say, if you were reading pixels row by row from a lat-long image map.) Conveniently, when we're accumulating samples from path tracing the sample directions are usually stratified or uniformly random. 

One note of caution: some low-discrepancy sequences (e.g. fixed-length ones like the Hammersley sequence) will not work well if successive samples are correlated, even though the sequence is well-distributed over the entire domain. 

The algorithm is as follows, and supports both non-negative and regular solves. While the implementation focuses on spherical Gaussians, the algorithm should work for any spherical basis function.

{% highlight swift %}
struct SphericalGaussianBasis {
    var lobes : [SphericalGaussian]
    var totalAccumulatedWeight : Float = 0.0
    var lobeMCSphericalIntegrals = [Float](repeating: 0.0, count: lobes.count) // Optional, can be precomputed at a slight increase in error.
    let nonNegativeSolve : Bool
    
    mutating func accumulateSample(_ sample: RadianceSample, sampleWeight: Float = 1.0) {
        totalAccumulatedWeight += sampleWeight
        let sampleWeightScale = sampleWeight / totalAccumulatedWeight
        
        var currentEstimate = float3(0)
        
        var sampleLobeWeights = [Float](repeating: 0.0, count: self.lobes.count)
        for (i, lobe) in self.lobes.enumerated() {
            let dotProduct = dot(lobe.axis, sample.direction)
            let weight = exp(lobe.sharpness * (dotProduct - 1.0))
            currentEstimate += lobe.amplitude * weight
            
            sampleLobeWeights[i] = weight
        }
        
        for i in 0..<self.lobes.count {
            let weight = sampleLobeWeights[i]
            if weight == 0 { continue }
            
            var sphericalIntegralGuess = weight * weight
            
            // The MC-computed spherical integral will be too inaccurate at first, so fall back to the
            // precomputed integral for the first iteration.
            if self.lobeMCSphericalIntegrals[i] == 0.0 {
                sphericalIntegralGuess = lobes[i].precomputedSphericalIntegral
            }
            self.lobeMCSphericalIntegrals[i] += (sphericalIntegralGuess - self.lobeMCSphericalIntegrals[i]) * sampleWeightScale
            
            // The most accurate method requires using the MC-computed spherical integral.
            // However, if you don't want to store a weight per-lobe you can instead substitute it with the
            // integral of the lobe over the domain at a slight increase in error.
            let sphericalIntegral = self.lobeMCSphericalIntegrals[i] // lobes[i].precomputedSphericalIntegral
            
            let otherLobesContribution = currentEstimate - self.lobes[i].amplitude * weight
            let newValue = (sample.value - otherLobesContribution) * weight / sphericalIntegral
            
            self.lobes[i].amplitude += (newValue - self.lobes[i].amplitude) * sampleWeightScale
            
            if (self.nonNegativeSolve) {
                self.lobes[i].amplitude = max(self.lobes[i].amplitude, float3(0))
            }
        }
    }
}
{% endhighlight %}

I've called this a 'running average' since each new sample is evaluated against the Monte-Carlo estimate of the function based on the previous samples. If we want each lobe to be non-negative, we simply clamp it; the next sample to come in will then be evaluated against the set of non-negative lobes. I'll post the full derivation of why this works in a follow-up blog post.

So how does it look? Well, here are the results for the 'ennis.hdr' environment map using Halton-sequence sample directions and twelve lobes, where 'Running Average' is my new method. In these images, I'm using [Stephen Hill's fitted approximation for a cosine lobe](https://mynameismjp.wordpress.com/2016/10/09/sg-series-part-3-diffuse-lighting-from-an-sg-light-source/) to evaluate the irradiance for all encoding methods. 

<table>
<tr><td><b>Radiance</b></td><td><b>Irradiance</b></td><td><b>Irradiance Error (sMAPE)</b></td><td><b>Encoding Method</b></td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceMCIS.png"/></td><td valign="top"><img src="/assets/spherical-gaussians/irradianceMCIS.png"/></td><td><center>N/A</center></td><td>Reference</td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceSG.png"/><br/>RMS: 4.24341</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceSG.png"/><br/>RMS: 0.44259</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceErrorSG.png"/></td><td>Naïve</td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceSGLS.png"/><br/>RMS: 3.81043</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceSGLS.png"/><br/>RMS: 0.241267</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceErrorSGLS.png"/></td><td>Least Squares</td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceSGRA2.png"/><br/>RMS: 3.80723</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceSGRA2.png"/><br/>RMS: 0.243617</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceErrorSGRA2.png"/></td><td>Running Average</td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceSGNNLS.png"/><br/>RMS: 3.93677</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceSGNNLS.png"/><br/>RMS: 0.808848</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceErrorSGNNLS.png"/></td><td>Non-Negative Least Squares</td></tr>
<tr><td valign="top"><img src="/assets/spherical-gaussians/radianceSGNNRA2.png"/><br/>RMS: 3.93653</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceSGNNRA2.png"/><br/>RMS: 0.807047</td><td valign="top"><img src="/assets/spherical-gaussians/irradianceErrorSGNNRA2.png"/></td><td>Non-Negative Running Average</td></tr>
</table>

It's a marked improvement over the naïve projection, and, depending on the sample distribution, can even achieve better results than a standard least squares solve. And it all works on the fly, on the GPU, requiring only a per-lobe mean and the total sample weight for all lobes to be stored. For best quality, I recommend also storing a per-lobe estimate of the spherical integral, as is done above; however, that can be replaced with the precomputed spherical integral at a small increase in error.

If you want to see this method running, Matt Pettineo has integrated it into [The Baking Lab](https://github.com/TheRealMJP/BakingLab). You can find it under the 'Running Average' and 'Running Average Non-Negative' solve modes.

The precomputed spherical integrals referenced in the code above are the integral of each basis function (e.g. each SG lobe) squared over the sampling domain. If the samples are distributed over a sphere then this has a closed-form solution; otherwise it can be precomputed using Monte-Carlo integration. Note that I've omitted dividing by the sampling PDF since the factors cancel out with the sampling in my algorithm above.

{% highlight swift %}
struct SphericalGaussian {
    var amplitude : float3
    var axis : float3
    var sharpness : Float
    
    var sphericalIntegral : Float {
        return (1.0 - exp(-4.0 * self.sharpness)) / (4 * self.sharpness)
    }
    
    var hemisphericalIntegral : Float {
        var total = 0.0 as Float
        let sampleCount = 2048
        
        for _ in 0..<sampleCount {
            let direction = float3.randomOnHemisphere
            
            let dotProduct = dot(self.axis, direction)
            let weight = exp(self.sharpness * (dotProduct - 1.0))
            total += weight * weight
        }
        
        return total / Float(sampleCount)
    }
}
{% endhighlight %}

While testing this, I used [Probulator](https://github.com/kayru/Probulator), a useful open-source tool for testing different lighting encoding strategies. The source code for the implementation of this method within Probulator is below.

Note that the original implementation of irradiance for the naïve projection in Probulator cheats a little bit since it uses a non-cosine BRDF to evaluate the spherical Gaussian lobes. If we're also using the spherical Gaussian to encode radiance for specular lighting, compensating with the BRDF doesn't really work; we want things to be consistent.

{% highlight c++ %}

class ExperimentSGRunningAverage : public ExperimentSGBase
{
public:
    
    void solveForRadiance(const std::vector<RadianceSample>& _radianceSamples) override
    {
        const u32 lobeCount = (u32)m_lobes.size();

        std::vector<RadianceSample> radianceSamples = _radianceSamples;
        // The samples should be uniformly randomly distributed (or stratified) for best results.
//                std::random_shuffle(radianceSamples.begin(), radianceSamples.end());

        float lobeMCSphericalIntegrals[lobeCount];

        for (u32 lobeIt = 0; lobeIt < lobeCount; ++lobeIt) {
            lobeMCSphericalIntegrals[lobeIt] = 0.f;
        }

        float lobePrecomputedSphericalIntegrals[lobeCount];
        for (u64 lobeIt = 0; lobeIt < lobeCount; ++lobeIt)
        {
            lobePrecomputedSphericalIntegrals[lobeIt] = (1.f - exp(-4.f)) / (4 * m_lobes[lobeIt].lambda);
        }

        float totalSampleWeight = 0.f;
        
        for (const RadianceSample& sample : radianceSamples) {
            const float sampleWeight = 1.f;
            totalSampleWeight += sampleWeight;
            float sampleWeightScale = sampleWeight / totalSampleWeight;

            vec3 currentEstimate = vec3(0.f);

            float sampleLobeWeights[lobeCount];
            for (u32 lobeIt = 0; lobeIt < lobeCount; ++lobeIt) {
                float dotProduct = dot(m_lobes[lobeIt].p, sample.direction);
                float weight = exp(m_lobes[lobeIt].lambda * (dotProduct - 1.0));
                currentEstimate += m_lobes[lobeIt].mu * weight;

                sampleLobeWeights[lobeIt] = weight;
            }

            for (u32 lobeIt = 0; lobeIt < lobeCount; ++lobeIt) {
                float weight = sampleLobeWeights[lobeIt];
                if (weight == 0.f) { continue; }

                float sphericalIntegralGuess = weight * weight;
                
                // The MC-computed spherical integral will be too inaccurate at first, so fall back to the
                // precomputed integral for the first iteration.
                if (lobeMCSphericalIntegrals[lobeIt] == 0.f) {
                    sphericalIntegralGuess = lobePrecomputedSphericalIntegrals[lobeIt];
                }
                
                lobeMCSphericalIntegrals[lobeIt] += (sphericalIntegralGuess - lobeMCSphericalIntegrals[lobeIt]) * sampleWeightScale;

                // The most accurate method requires using the MC-computed spherical integral.
                // However, if you don't want to store a weight per-lobe you can instead substitute it with the
                // (potentially analytically computed) integral of the lobe over the domain at a slight increase in error.
                float sphericalIntegral = lobeMCSphericalIntegrals[lobeIt];

                vec3 otherLobesContribution = currentEstimate - m_lobes[lobeIt].mu * weight;
                vec3 newValue = (sample.value - otherLobesContribution) * weight / sphericalIntegral;
                
                m_lobes[lobeIt].mu += (newValue - m_lobes[lobeIt].mu) * sampleWeightScale;

                if (m_nonNegativeSolve) {
                    m_lobes[lobeIt].mu = max(m_lobes[lobeIt].mu, vec3(0.f));
                }
            }
        }
    }
};

{% endhighlight %}
