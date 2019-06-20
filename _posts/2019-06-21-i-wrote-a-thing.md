---
layout: post
title:  "So, I Wrote a Master's Thesis"
date:   2019-06-21 11:08:26 +1200
categories: rendering lightmaps path-tracing thesis
---

My Master's Thesis – Interactive Generation of Path-Traced Lightmaps – is finally [publicly available](/thesis)! I spent a year writing and working on it, and I really hope that other people can find some use from it. 

It can be rather dense in parts (an unfortunate side-effect of the word limit), so please feel free to contact me (see the page footer) if you have any questions – I can help explain any part of it in more detail.

I also plan/hope to extract parts of it out into longer, more casually-written blog posts – watch for that in the future. The first candidate for that is likely to be the [Ambient Dice](https://www.activision.com/cdn/research/ambient_dice_web.pdf) specular reconstruction – it's surprisingly easy to achieve better quality and performance than e.g. spherical Gaussian lightmaps for specular by using other basis functions and approximations, and I'm sure I've barely scratched the surface of what could be done.

Finally, you may notice my [CV and Portfolio](/portfolio.md) has been updated, and that I'm looking for work. If you have any remote work, particularly in low-level, graphics, or engine development, then I'd love to hear from you. I'm based in Wellington, New Zealand, which puts me in a friendly time zone to Australia, much of Asia, and the west coast of the US. I'm also open to full-time employment in Wellington starting from September or October.

*[Crytek Sponza](https://www.cryengine.com/marketplace/sponza-sample-scene) rendered using a hemispherical Ambient Dice lightmap (a technique developed for the thesis) for diffuse and specular*:

![Sponza rendered using a hemispherical Ambient Dice lightmap for diffuse and specular](/assets/thesis/Sponza-LookingUp-AmbientDiceHemi.jpg)

*Path-traced reference:*

![Sponza rendered from the same perspective by a path tracer](/assets/thesis/Sponza-LookingUp-PathTracedReference.jpg)