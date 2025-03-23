---
layout: post
title: "On Subsurface Parallax Shading"
categories: rendering
---

Recently I tried implementing a fake depth parallax shader. The basic idea I had is to make it look like the texture given is some depth $D$ units below the surface. The surface itself could vary in refractive index, maybe to make it look like it's encased in glass or resin.

First I started by drawing a diagram like this,

<p style="text-align: center;">
	<img src="/assets/img/subsurface_diagram.png" width="600">
</p>

What we're looking for is that little offset $\mathbf{u}$.

Since we're working in terms of UV coordinates on the surface, it's helpful to think in terms of tangent space, so we have to transform our view direction by the inverse of the TBN matrix. As it's orthonormal, its inverse is its transpose.

$$
\mathbf{v} = \text{TBN}^\text{T} \cdot \text{worldspace view direction}
$$

I want to stress now that there *is* a reason why model importers like **[Assimp](https://assimp.org/)**, when told to `aiProcess_CalcTangentSpace`, generate all three components of the TBN matrix for each vertex. While yes, you can technically calculate the third component by just computing the cross product of the first two, this doesn't gurantee that the third component will point in the direction of increasing UV, which is necessary for the conversion to UV space to work. I worked on this for like a day until I realised that. Sigh...

Working in tangent space means that $\mathbf{n} = (0, 0, 1)$, so $(\mathbf{n} \cdot \mathbf{v}) = v_3 = \cos \theta_o$. Now $\mathbf{u}$ can be found by doing some basic trigonometry,

$$
\mathbf{u} = -\frac{D}{v_3} \cdot \mathbf{v}_{12}
$$

If we want to do refraction it's as simple as refracting the view direction and then using that in the final $\mathbf{u}$ calculation,

$$
\mathbf{v}' = \text{refract}(\mathbf{v}, \mathbf{n}, \eta)
$$

So our parallax code looks like,

{% highlight hlsl %}
float2 subsurfaceParallax(float2 uv, float3 viewDir, float eta, float depth)
{
	float2 refractedViewDir = refract(-viewDir, float3(0.0, 0.0, 1.0), eta);
	float2 u = depth / refractedViewDir.z * refractedViewDir.xy;
	return uv - u;
}
{% endhighlight %}

We can then use this to sample our texture with the new UV coordinates as usual.

<!-- enable latex -->
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js" async></script>
<script type="text/javascript">MathJax={tex:{inlineMath:[['$','$']]}};</script>
