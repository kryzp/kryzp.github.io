---
title: "Screen Space Compute Particles based on Naughty Dog SigGraph"
date: 2025-03-28
draft: true
---

I watched a wonderful [SigGraph video](https://www.youtube.com/watch?v=_bbPeCwNxAU) by the devs at Naughty Dog that talked about how their screen space GPGPU-driven particle system works. Naturally, I really wanted to implement something like that on my own to better understand how it works / how to implement it, any limitations, and generally to just create some nice looking scenes.

The core idea is that particle systems don't need super accurate physics simulation so instead you can use the depth buffer to do collision testing. Furthermore, if you do things like render material ID's to a sepereate buffer, combined with using a `ddx` and `ddy` trick to compute normals from the depth buffer (regular normal buffer has too many details) you can do really fast "close enough" intersection physics, and even details like making snow particles melt quicker on skin rather than clothing.

The idea is we can calculate the screen space coordinate of the particle we are simulating. Then, we 

$\Sigma_k : \mathrm{R}^3 \to \mathrm{R}^2$ which maps world space positions into screen space, and another function $\Lambda_k : \mathrm{R}^2 \to \mathrm{R}^3$ which maps from screen space into world space by sampling the depth buffer. The $k$ corresponds to what frames view matrices we're going to be using.
