---
layout: page
title: CV and Portfolio
permalink: /portfolio/
---

## Biography

I'm Thomas Roughton, a programmer, dabbling designer, hobbyist musician and photographer, and musical theatre enthusiast. I'm based in Wellington, New Zealand, and you can hire me for game, graphics, or engine work. I can work on-site in Wellington or remotely, and can adapt to a wide range of development practices and tech stacks.

I specialise in computer graphics and real-time rendering, where I've contributed techniques such as [progressive least squares encoding](/rendering/irradiance-caching/spherical-gaussians/2018/09/21/spherical-gaussians). I have expert experience with the Metal rendering API, have experience with OpenGL, and have also written a Vulkan backend for [my render-graph based rendering framework](https://github.com/troughton/SwiftFrameGraph).

I am highly motivated, have high standards, and learn quickly. I'm constantly exploring possibilities, finding ways that things could be done better, and acting upon them. I strongly value the end user experience in my work, whether that be a consumer, artist, or API user, and am excited more by what technology enables people to do than by the technology itself.

My background in programming began with writing iOS apps in Objective-C at the age of 13. Since 2015, I've been writing most of my code in the [Swift programming language](https://swift.org), and have contributed to the compiler in order to help port it to Windows. I also have significant experience with C, C++, Java, C#, and Python. The promise of languages like Swift and Rust excites me, and I'm looking forward to seeing the industry move away from C and C++. 

Over the course of my University studies, I've worked in conjunction with [Joseph Bennett](http://josephbennett.me) to develop a cross-platform game engine in Swift. This has been a great opportunity to learn about a wide range of topics, including engine architecture, data-oriented design, different graphics APIs, various rendering techniques, allocator management, SIMD programming, and more. We hope to use this engine to release a game in the near future.

## Qualifications

- Master of Science in Computer Science (Victoria University of Wellington, 2019, pending acceptance): Interactive Generation of Path-Traced Lightmaps.
- Victoria Masters by Thesis Scholarship (2018)
- Bachelor of Science, majoring in Computer Science and minoring in Media Design (A+ average) (Victoria University of Wellington, 2016).
- Victoria Vice Chancellor's Excellence Scholarship (2014)
- NZQA Outstanding Scholarships in Calculus and Classics, and Scholarships in Physics, Chemistry, and Technology.

## Work Experience

### Software Development

- Internship at [Weta Digital](https://www.wetafx.co.nz) (November 2017 – February 2018), working on skin rendering. I researched and implemented a novel solution for thick- and thin-surface subsurface scattering within their in-house OpenGL preview renderer Gazebo, meeting the constraints of their production pipeline.

- Full-time summer work at [TouchTech](https://touchtechlabs.com/) (January – February 2015) as an iOS engineer. I extended the iPad version and led development on the iPhone version of the [UBank app](https://itunes.apple.com/us/app/ubank-mobile-banking/id1313119623), and was brought on to the [Ngā Tapuwae Gallipoli app](https://itunes.apple.com/us/app/ngā-tapuwae-western-front/id1039180827) to address bugs and UI issues. Despite my short time in this role, I quickly became considered an expert in the team and was referred and solved many difficult problems that the team had been struggling with.

- Sole developer of [CubeTimer](https://www.speedsolving.com/threads/cubetimer-for-ios-2-0.39187/) (2011 – 2012), an iOS speedcubing application. This started as a hobby when some friends wanted to time how quickly they could solve Rubik's cubes, but quickly expanded into a fully-featured application.

I also contribute to the open-source [Swift programming language](https://github.com/apple/swift) and actively develop my own graphics framework called [SwiftFrameGraph](https://github.com/troughton/SwiftFrameGraph). You can find some of my code [on GitHub](https://github.com/troughton/), in varying states of production-readyness.

### Web Development

- [Orb Solutions](https://orbsolutions.co.nz) (2018), adapting an external designer's concepts into a Squarespace template.

- [Dragonfly Hair Design](http://www.dragonflyhairdesign.co.nz) (2012), designing and building a clean HTML + CSS website to match their brand.

## Portfolio

### 'Interdimensional Llama' (2016)

![Interdimensional Llama](/assets/portfolio/IDLHeader.png)

Interdimensional Llama was a perspective-switching puzzle game made in conjunction with Phoebe Zeller, Casey Garnock-Jones, Joseph Bennett, Jiaheng Wang, and Cameron Hopkinson. You play as Lou, a llama with the ability to switch between 2D and 3D to navigate obstacles in the world and get to the goal. 

Along with helping to design the rules for the game, I worked on C# gameplay code within Unity, primarily contributing to the grid-based navigation system and camera management code.

#### 'Atmospheric Llama' (2017)

![Atmospheric Llama](/assets/portfolio/AtmosphericLlama.jpg)

As part of a university course, the task was to select and implement a modern graphics technique; I chose temporal antialiasing, and worked with [Joseph Bennett](http://josephbennett.me), who implemented frustum-based volumetrics.

<iframe width="740" height="416" src="https://www.youtube-nocookie.com/embed/Nlr7m4rq37A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

_(I recommend watching the video at 1.5x - 2x speed.)_

To add some extra challenge, we decided to implement this within our own engine, which was written more-or-less from the ground up for this project. This ended up including implementing an early version of [SwiftFrameGraph](https://github.com/troughton/SwiftFrameGraph), a layer-based animation system, local light probes, clustered shading, and BVH-based frustum culling, all in the space of twelve part-time weeks. There are many technical issues in the final result – directional shadows are particularly rough – but I'm still proud of what we managed to achieve.

### 'Official Business' (2017)

For this university project, the task was to composite synthetic objects into a real scene using only a modified version of [PBRT](https://pbrt.org) (i.e. no Photoshop adjustments). I implemented [differential rendering](http://www.pauldebevec.com/Research/IBL/debevec-siggraph98.pdf) within PBRT, constructed proxy geometry to match the photographed scene, and tweaked materials.

The below image contains four synthetic objects. See if you can spot them!

![Official Business with digital objects](/assets/portfolio/OfficialBusiness.jpg)

<details><summary>Show Spoilers</summary>

<br>
<p>Source image:</p>

<p><img src="{{ site.baseurl }}/assets/portfolio/OfficialBusinessUnmodified.jpg"></p>

<p>Proxy Geometry:</p>
 
<p><img src="{{ site.baseurl }}/assets/portfolio/OfficialBusinessProxyGeometry.jpg"></p>

I modelled the standing desk, wooden desk, and red audio box in this image; all other objects are licensed  assets.

</details><br>

### 'Illumination' (2016)

<iframe src="https://player.vimeo.com/video/183785457" width="740" height="416" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>

Illumination is a semi-procedural realtime animation driven by a music track I created. The MIDI events for the music were exported from my DAW and imported into our custom-built 3D engine, which then used it to apply scripted animation and lighting changes to a scene imported from Maya. The animation makes use of physically-based rendering and [LTC](https://eheitzresearch.wordpress.com/415-2/)-based polygonal area lights.


### Modelling and Animation

All of the projects below were done within the span of fourteen weeks as part of a part-time University course. 

#### 'Waterfall Swoop' (2015)

![Waterfall Swoop](/assets/portfolio/WaterfallSwoop.jpg)

Waterfall Swoop was the first project in a modelling and animation course. I modelled the hawk, used Maya particles to generate a waterfall, and then animated the scene to achieve the motion blur in the final render.

![Waterfall Swoop Hawk Model](/assets/portfolio/WaterfallSwoopModel.png)

#### 'March to Scurry' (2015)

<iframe src="https://player.vimeo.com/video/339451361" width="740" height="416" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>

Our task for this project was to take a model someone else had made (in my case, an ant) and transform it into a different model; I chose a living computer mouse. I modelled everything apart from the ant in this scene and animated it all.

#### 'Spring' (2015)

![Spring Poster](/assets/portfolio/Spring.jpg)

For this project, we had to design and animate our own short film. Spring is about a greenhouse robot trying to survive the winter.

<iframe src="https://player.vimeo.com/video/339454572" width="740" height="416" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>

Spring was an interesting challenge due to my unwittingly creating a very geometrically complex scene to render. The last few weeks of the project were an attempt to scale down the scene complexity to the point where I could meet the project deadline; as such, the rendering quality suffers in a few places and I wasn't quite able to get the level of polish I wanted.
