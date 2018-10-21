---
layout: post
title:  "Running Average Encoding - Why It Works"
date:   2018-10-02 11:51:00 +1300
categories: rendering irradiance-caching spherical-gaussians
---

In the [previous post](/rendering/irradiance-caching/spherical-gaussians/2018/09/21/spherical-gaussians.html) I introduced an as-far-as-I-know novel method for performing progressive least squares optimisation with spherical basis functions. Here, I'll go into more detail about how it works, and also derive my [original, approximate method](/rendering/irradiance-caching/spherical-gaussians/2018/09/21/spherical-gaussians-old.html) from the corrected version.

Many thanks to Peter-Pike Sloan for [providing the first part](https://twitter.com/PeterPikeSloan/status/1044482721223856128) of this derivation.

We'll be dealing with spherical integrals for the sake of this post, but everything is equally applicable to hemispheres by restricting the integration domain. For example:

$$ \int_S f(s) $$

will be used as shorthand for 'the integral over the sphere of the function $$ f(s) $$, where $$ s $$ is a direction vector. All integrals will be done in respect to $$ s $$.

$$ f(s) $$ is taken to mean the value of the function we're trying to fit in direction $$ s $$; this value will usually be obtained using Monte Carlo sampling.

We'll also assuming fixed basis functions parameterised only by their direction, such that $$ B_i(s) $$ is the value of the *i*th basis function in direction $$s$$. The basis functions will be evaluated by multiplying with a per-basis amplitude $$ b_i $$ and summing, such that the result $$ R $$ in direction $$ s $$ is given by:

$$ R(s) = \sum_i b_i B_i(s) $$

In the case of spherical Gaussians, $$ b_i = \mu_i $$, and $$ B_i(s) = e^{\lambda_i (s \cdot \vec{p}_i - 1)} $$, or the value of the *i*th lobe evaluated in direction $$ s $$. 

Our goal is to minimise the squared difference between $$ R(s) $$ and $$ f(s) $$ so that the fit matches the original function as closely as possible. Mathematically, that can be expressed as:

$$ \min \int_S ( \sum_i b_i B_i(s) - f(s))^2 $$

To minimise, we differentiate the function with respect to each unknown $$ b_i $$ and then set the derivative to 0.

$$ E = \int_S ( \sum_i b_i B_i(s) - f(s))^2 $$

$$ \frac{dE}{b_i} = 0 $$

Let $$ g(s) = \sum_k b_k B_k(s) - f(s) $$. Therefore, $$ \frac{d}{b_k} \begin{bmatrix} g(s) \end{bmatrix} = B_k(s) $$ for each $$ b_k $$.

$$
\begin{align*}

\frac{dE}{b_i} &= \frac{d}{b_i} \begin{bmatrix} \int_S ( \sum_i b_i B_i(s) - f(s))^2 \end{bmatrix} \\ 
               &= \frac{d}{b_i} \begin{bmatrix} \int_S ( g(s) )^2 \end{bmatrix} \\ 
               &= 2 \int_S g(s) \frac{d}{b_i} \begin{bmatrix} g(s) \end{bmatrix}  \\ 
               &= 2 \int_S g(s) B_i(s)  \\ 
              &= 2(  \sum_k b_k \int_S ( B_i(s) \cdot B_k(s) ) ) - 2 \int_S (B_i(s) \cdot f(s))
\end{align*}
$$

Therefore, by setting $$ \frac{dE}{b_i} = 0 $$,

$$
\begin{equation} \label{LeastSquaresMinimiser}
\int_S (B_i(s) \cdot f(s)) = \sum_k b_k \int_S ( B_i(s) \cdot B_k(s) ) ) 
\end{equation}
$$

At this step, we now have a method for producing a Monte Carlo estimate of the raw moments $$ B_i(s) \cdot f(s) $$: as each sample comes in, multiply it by each basis function and add it to the estimate for each lobe. This is in fact what was done for the naïve projection [used in The Order: 1886]((https://mynameismjp.wordpress.com/2016/10/09/sg-series-part-5-approximating-radiance-and-irradiance-with-sgs/)). To reconstruct the lobe amplitudes $$ b_i $$ we need to multiply by the inverse of $$ \int_S ( B_i(s) \cdot B_k(s) ) ) $$:

$$
\begin{bmatrix}
           b_{1} \\
           b_{2} \\
           \vdots \\
           b_{m}
         \end{bmatrix}
         =
         \begin{pmatrix}
           \int_S ( B_1(s) \cdot B_1(s) ) & \int_S ( B_1(s) \cdot B_2(s) ) & \dots & ( B_1(s) \cdot B_m(s) ) \\
           \int_S ( B_2(s) \cdot B_1(s) ) & \int_S ( B_2(s) \cdot B_2(s) ) & \dots & ( B_2(s) \cdot B_m(s) ) \\
           \vdots & \vdots & \ddots & \vdots  \\
           \int_S ( B_m(s) \cdot B_1(s) ) & \int_S ( B_m(s) \cdot B_2(s) ) & \dots & ( B_m(s) \cdot B_m(s) )
         \end{pmatrix}^{-1}
         \begin{bmatrix}
           \int_S (B_1(s) \cdot f(s)) \\
           \int_S (B_2(s) \cdot f(s)) \\
           \vdots \\
           \int_S (B_m(s) \cdot f(s))
         \end{bmatrix}
$$

This is a perfectly valid method of performing least squares without storing all of the samples at every step, although it [can be noisier](https://twitter.com/PeterPikeSloan/status/1044787585900400640) than if all samples were used to perform the fit. However, it does require a large matrix multiplication to reconstruct the $$ b_i $$ amplitudes, which is unsuitable for progressive rendering.

In the 'running average' algorithm, we want to reconstruct the $$ b_i $$ amplitudes as every sample comes in so that the results can be displayed at every iteration. There are therefore a few more steps we need to perform.

Let's rearrange the above equation to solve for a single $$ b_i $$.

$$ 
\begin{align*}
\int_S ( B_i(s) \cdot f(s) ) &= \sum_k b_k \int_S ( B_i(s) \cdot B_k(s) ) ) \\
                             &= b_i \int_S B_i(s)^2 + \sum_{k, k \not= i} b_k \int_S ( B_i(s) \cdot B_k(s) )
\end{align*}
$$

$$ 
b_i \int_S B_i(s)^2 = \int_S ( B_i(s) \cdot f(s) ) - \sum_{k, k \not= i} b_k \int_S ( B_i(s) \cdot B_k(s) )
$$

We can bring the entire right hand side under the same integral due to the linearity of integration.

$$
\begin{align*}
b_i \int_S B_i(s)^2 &= \int_S ( B_i(s) \cdot f(s)  - \sum_{k, k \not= i} b_k ( B_i(s) \cdot B_k(s) ) ) \\
                      &= \int_S ( B_i(s) \cdot ( f(s)  - \sum_{k, k \not= i} b_k \cdot B_k(s) ) )
\end{align*}
$$

Finally, we end up with the following equation for $$ b_i $$:

$$
b_i = \frac{\int_S ( B_i(s) \cdot ( f(s)  - \sum_{k, k \not= i} b_k \cdot B_k(s) ) )}{\int_S B_i(s)^2 }
$$

The two spherical integrals here which can be computed in tandem using Monte-Carlo integration. The estimate for $$ b_i $$ given a single sample in direction $$ \omega $$ with a value $$ v $$ (where $$ v $$ is an estimate of $$ f(\omega) $$) is given by:

$$
b_i = \frac{B_i(\omega) \cdot ( v - \sum_{k, k \not= i} b_k \cdot B_k(\omega) )}{\int_S B_i(s)^2 }
$$

The average value of $$ b_i $$ across all samples will tend towards the true least-squares value.

Likewise, the estimator for $$ \int_S (B_i(s))^2 $$ is given by averaging $$ B_i(\omega)^2 $$.

To solve this for $$ b_i $$, we need to know the amplitudes $$ b_k $$ for all $$ k $$ where $$ k \not= i $$. We can approximate this by using the $$ b_k $$ values solved for in the previous iteration of the algorithm. As the number of samples increases, the $$ b $$ vector will gradually converge to the true value. The convergence could potentially be improved by seeding the $$ b $$ vector with the estimate from a low-sample-count run rather than with the 0 vector; in practice, the error seems to disappear fairly quickly.

Similarly, since $$ v $$ is often only an estimator for the function value $$ f(\omega) $$ and not the true value, high variance in its estimate can cause errors in the $$ b $$ vector. One possible strategy to counter this is to gradually increase the sample weights over time (e.g. with $$ w = 1 - exp(-\frac{sampleIndex}{sampleCount}) $$); however, in my implementation I haven't found this to be necessary.

In this running average method, the integral in the denominator is calculated using Monte Carlo integration in the same way that $$ b_i $$ is. In fact, it turns out that computing both of them in lockstep improves the accuracy of the algorithm since any sampling bias in the numerator will be partially balanced out by the bias in the denominator. However, it's also true that the integral may be wildly inaccurate at small sample counts and end up amplifying small values; therefore, to balance that out, I recommend clamping the estimator for the integral to at least the true integral. Alternatively, it's possible to always use the precomputed true integral on the denominator and only estimate the $$ b $$ vector, although this results in slightly increased error.

---------------
<br>

My [original algorithm](/rendering/irradiance-caching/spherical-gaussians/2018/09/21/spherical-gaussians-old.html) was created by experimentation. I thought it would be worth going through why it worked and the approximations it made. Note that none of this is necessary to understand the corrected equation – it's purely for curiosity and interest!

Effectively, at each step, it solved the following equation:

$$
b_i = \frac{ B_i(\omega) \cdot (v - \sum_k b_k B_k(\omega) ) } { \int_S ( B_i(s) ) } + b_i
$$

If we rearrange that to get into a form vaguely resembling our proper solution above:

$$
\begin{align*}
b_i &= \frac{ B_i(\omega) \cdot (v - \sum_k b_k B_k(s) ) } { \int_S ( B_i(s) ) } + b_i \\
    &= \frac{ B_i(\omega) \cdot (v - \sum_k b_k B_k(s) ) + b_i \int_S B_i(s) } { \int_S B_i(s)  } \\
    &\approx \frac{ B_i(\omega) \cdot (v - \sum_k b_k B_k(s) ) + b_i B_i(\omega) } { \int_S B_i(s) } \\
    &\approx \frac{ B_i(\omega) \cdot (v - \sum_k b_k B_k(s) + b_i ) } { \int_S B_i(s) } \\
    &\approx \frac{ B_i(\omega) \cdot (v - \sum_{k, k \not= i} b_k B_k(\omega) - b_i B_i(\omega) + b_i ) } { \int_S B_i(s) } \\
    &\approx \frac{ B_i(\omega) \cdot (v - \sum_{k, k \not= i} b_k B_k(\omega) + (1 - B_i(\omega)) b_i) } { \int_S B_i(s) }
\end{align*}
$$

For the spherical integral of a spherical Gaussian basis function with itself, $$ \int_S (B_i(s)) \approx 2 \int_S (B_i(s))^2 $$, since $$ \int_S (B_i(s))^2 = \frac{\pi}{4 \lambda} (1 - e^{-4 \lambda}) $$ and $$ \int_S B_i(s) = \frac{\pi}{2 \lambda} (1 - e^{-2 \lambda}) $$. Therefore,

$$
\begin{align*}
b_i &\approx \frac{ B_i(\omega) \cdot (v - \sum_{k, k \not= i} b_k B_k(\omega) + (1 - B_i(\omega)) b_i) } { 2 \int_S (B_i(s))^2 } \\
&\approx \frac{ B_i(\omega) \cdot (v - \sum_{k, k \not= i} b_k B_k(\omega))} { 2 \int_S (B_i(s))^2 } + \frac{ B_i(\omega) \cdot (1 - B_i(\omega)) b_i) } { 2 \int_S (B_i(s))^2 }
\end{align*}
$$

This is very close to our 'correct' equation above. In fact, it becomes equal when

$$
v - \sum_{k, k \not= i} b_k B_k(\omega) = (1 - B_i(\omega))b_i
$$

We can rearrange that a little further:

$$
\begin{align*}
v &= b_i - b_i B_i(\omega) + \sum_{k, k \not= i} b_k B_k(\omega) \\
     &= b_i + \sum_{k} b_k B_k(\omega) - 2 b_i B_i(\omega)
\end{align*}
$$

Since $$ v $$ is an estimator for $$ f(\omega) $$ and we assume that, as the fit converges, $$ f(s) \approx \sum_{k} b_k B_k(s) $$, we're left with:

$$
2b_i B_i(\omega) = b_i \\
B_i(\omega) = \frac{1}{2}
$$

In other words, using the original algorithm for a given sample, the error is mostly determined by how close $$ B_i(s) $$ is to $$ \frac{1}{2} $$. Since the influence of samples with higher basis weights $$ B_i(s) $$ is greater anyway, this turned out to be a reasonable approximation. However, given the option, I'd still recommend using the corrected algorithm!

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=default.js">
</script>