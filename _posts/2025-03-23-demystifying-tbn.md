---
layout: post
title: "Demystifying the TBN matrix"
categories: rendering
---

One thing that seemed really abstract to me when I first learned about normal mapping is the TBN matrix. I mean, I kind of understood it, it converted from tangent space into world space, simple enough. It meant that we could take our generic tangent space normal maps, and attach them to whatever surface we wanted.

That being said, I'd be lying if that meant I actually had a proper picture for what it was.

I think that a better way to look at it is that it's something that describes what the surface the fragment you're shading looks like, giving you the direction along the surfaces' UV and the direction directly vertically out of the surface.

<p style="text-align: center;">
	<img src="/assets/img/tbn_sphere.png" width="400">
</p>

Being able to convert world coordinates into a "local" form of space relative to the pixel itself is really helpful for, e.g: converting world space into a uv offset. Since the TBN matrix is "orthonormal", its inverse is its transpose, so converting between the two spaces is super cheap.

<!-- enable latex -->
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js" async></script>
<script type="text/javascript">MathJax={tex:{inlineMath:[['$','$']]}};</script>
