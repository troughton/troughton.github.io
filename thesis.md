---
layout: page
title: Master's Thesis
permalink: /thesis
---

## Interactive Generation of Path-Traced Lightmaps

[Download Here](/Thesis.pdf) (11MB PDF)

| ![Sponza rendered using a hemispherical Ambient Dice lightmap for diffuse and specular](/assets/thesis/Sponza-LookingUp-AmbientDiceHemi.jpg) | ![Sponza rendered from the same perspective by a path tracer](/assets/thesis/Sponza-LookingUp-PathTracedReference.jpg) |
| [Crytek Sponza](https://www.cryengine.com/marketplace/sponza-sample-scene) rendered using a hemispherical Ambient Dice lightmap for diffuse and specular | Path-traced reference |

### Abstract

Indirect illumination is an important part of realistic images, and accurately simulating the complex effects of indirect illumination in real-time applications has long been a challenge for the industry. One popular approach is to use offline precomputed solutions such as *lightmaps* (textures containing the precomputed lighting in a scene) to efficiently approximate these effects. Unfortunately, these offline solutions have historically enforced long iteration times that come at a cost to artist productivity. These solutions have additionally either supported only the low-frequency diffuse component of indirect lighting, yielding poor visual results for glossy or metallic materials, or have used overly expensive approximations.

In recent years, the state of the art lightmap precomputation pipeline has shifted to using highly vectorised path tracing, often on GPU hardware, to compute the indirect illumination effects. The use of path tracing enables *progressive rendering*, wherein an approximation to the full solution is found and then refined as opposed to solving for the final result in a single step. Progressive rendering through path tracing thereby helps to provide rapid iteration for artists.

This thesis describes a system that can progressively path-trace indirect illumination lightmaps on the GPU. Contributing to this system, it introduces a new gather-based method for sample accumulation, enhances algorithms from prior work, and presents a range of encoding methods, including a novel progressive method for non-negative least-squares encoding of spherical basis functions. 

In addition, it presents a novel, efficient solution for high-quality precomputed diffuse and low-frequency specular indirect illumination that extends the Ambient Dice family of spherical basis functions. This solution provides comparable or better specular reconstruction to prior work at lower runtime cost and has potential for widespread use in real-time applications.

### Chapter Listing

- **Chapter 1: Introduction**

- **Chapter 2: Background** includes a general introduction to lightmaps, summarises related work, and discusses different possible techniques for lightmap baking. Additionally, it provides an introduction to GPU SIMD architectures, summarises wavefront path tracing as a method of formulating path tracing for the GPU, and outlines GPU radix sort.

- **Chapter 3: Architecture of the Lightmap Renderer** provides a basic framework for the lightmap path tracing renderer, then introduces a filtered accumulation method, an adaptive rendering method, alterations to support lightmap baking, and finally a method of implementing camera-based lightmap sampling.

- **Chapter 4: Lightmap Parameterisation** describes practical considerations of the method [used within EA's Frostbite](https://www.ea.com/frostbite/news/precomputed-global-illumination-in-frostbite) for parameterising geometry into lightmaps, including performance metrics for different implementations and a simple means of compression necessary for copying the parameterisation for use in GPU path tracing. 

- **Chapter 5: Accelerating the Path Tracer: Improving Coherence** discusses how stream compaction can be integrated into the path tracing framework, describes how GPU SIMD operations can accelerate radix sort on the GPU, evaluates ray direction sorting as an acceleration method, and introduces a tile-based indexing method that helps to achieve coherence by grouping similar rays.

- **Chapter 6: Accelerating the Path Tracer: Reducing Variance** overviews next event estimation, progressive sample sequences, and importance sampling as methods of variance reduction. It also discusses how biased light sampling or irradiance caching can more quickly produce an image at the cost of bias.

- **Chapter 7: Spherical Basis Functions** delves into spherical basis functions as a method for encoding radiance or irradiance, providing the mathematical formalism for least-squares encoding of any general set of linear basis functions. That formalism is then extended with a novel method for progressive least-squares and non-negative least-squares encoding of spherical basis functions in an efficient and simple-to-implement manner. 

- **Chapter 8: Families Spherical Basis Functions** overviews the spherical harmonic, spherical Gaussian, and Ambient Dice families of basis functions. For Ambient Dice, the cosine-lobe variant is extended to provide diffuse and specular reconstruction from encoded radiance lobes. This reconstruction is then compared with spherical Gaussians and spherical harmonics, demonstrating its applicability as an efficient encoding format for low-frequency lighting.

- **Chapter 9: Conclusion**

- **Appendices**

    - **Appendix A: Validation and PBRT Compatibility** briefly discusses how the path tracer was validated against the rasteriser and the [open source PBRT renderer](https://pbrt.org/).

    - **Appendix B: Implementation Frameworks** gives an overview of the custom 3D engine LlamaEngine and the [SwiftFrameGraph](https://github.com/troughton/SwiftFrameGraph) rendering framework, which serve as the base implementation frameworks for the thesis. It also includes technical details pertaining to integrating the [RadeonRays](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonRays_SDK) and [Metal Performance Shaders](https://developer.apple.com/documentation/metalperformanceshaders) ray-tracing frameworks into the pre-existing engine.

    - **Appendix C: Image Gallery: Reconstruction Error from Ambient Dice vs. Spherical Gaussians** provides a detailed image comparison of the Ambient Dice and spherical Gaussian encoding techniques and reconstruction quality from each on a range of environment maps.

    - **Appendix C: Image Gallery: Test Scenes** provides images of a number of test scenes referenced within the thesis.


----

<br>
<a rel="license" href="http://creativecommons.org/licenses/by-nd/3.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nd/3.0/88x31.png" /></a>

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nd/3.0/">Creative Commons Attribution-NoDerivs 3.0 Unported License</a>.