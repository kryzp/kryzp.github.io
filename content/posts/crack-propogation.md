---
title: "Crack Propogation Simulation"
date: 2025-06-01
---

I recently watched [this](https://www.youtube.com/watch?v=0ebPkjqV7jE) video by Alexander Sannikov which was part of a larger Path of Exile rendering talk and found it pretty interesting. I figured that as a small weekend project I'll try to implement something similar because I'm bored.

The basic idea is that you create a triangular mesh of points, with the edges being ideal springs (ones which abide Hookes' Law) that slowly reduce their natural length over time until they hit a minimum natural length, to simulate drying. This is called a "quasistatic" simulation because it is assumed to happen so slowly that all inertial effects are negligible. To make sure all springs don't just all break at once, we slightly vary the spring constant and breaking tensions per spring to emulate microscopic impurities.

# Model Definition

Each point has a position $\mathbf{x}$, velocity $\mathbf{v}$, sum of forces $\mathbf{F}$, mass $m$, and a toggle to determine if it's fixed or not.

Each spring has a natural length $L$, a spring constant $k$, a breaking strain $\varepsilon_T$, and a toggle to determine if it's dead.

Finally the simulation itself has a couple of parameters,
 - Minimum spring length factor $\alpha$
 - Velocity dampening factor $\mu$
 - Shrinking rate $s$
 - Bound values for $k$ and $\varepsilon_T$ when randomly assigning them to springs

# Creating the mesh

Since the mesh is perfectly regular we don't need to use any super complicated triangulation libraries or anything. Instead, I just create a regular rectanular mesh except each layer the direction of the diagonal edge gets flipped. Then every other layer just gets shifted half the `dx` to the right.

{{< image src="/assets/img/crack_propogation_mesh.png" position="center" style="border-width: 1px; width: 500px;" >}}

# Simulation

First, we go through all springs and apply their forces to the connected points. To do this, we calculate the length $\ell$ between the two connected points, and then calculate the strain as,

$$
\varepsilon = \frac{\ell - L}{L}
$$

If $\varepsilon \geq \varepsilon_T$, then we mark the spring as "dead" and exit there, this spring is no longer simulated. Otherwise, we calculate the force magnitude between the two points as,

$$
F = k \cdot (\ell - L)
$$

And then add the corresponding vectors to the force accumulator $\mathbf{F}$ for both points. Then, we shrink the springs natural length to emulate drying,

$$
L_{n+1} = L_n + \Delta{t} \cdot s(\alpha L_0 - L_n)
$$

You could use multiple different models for how the natural length shrinks over time but this is the one I chose. Making sure the spring length is properly capped is crucial to preventing the simulation from completely collapsing into itself.

Finally, we iterate through all points (that aren't fixed) and move them using semi-implicit Euler,

$$
\begin{aligned}
\mathbf{v}_{n+1} &= \mu \left( \mathbf{v}_n + \Delta{t} \cdot \frac{\mathbf{F}_n}{m} \right) \\\\
\mathbf{x}_{n+1} &= \mathbf{x}_n + \Delta{t} \cdot \mathbf{v}_{n+1} \\\\
\mathbf{F}_{n+1} &= \mathbf{0} \ \text{(we set the force accumulator back to zero)}
\end{aligned}
$$

Notice that since this is a quasistatic simulation we don't account for inertial effects (we assume acceleration is negligible) so acceleration is never directly kept track of.

# Summary

I wrote it in C++ using SFML and the whole thing fits into a couple screens of code. There's plenty more that could be done (e.g: using GPGPU for simulation, rendering with instancing, etc...) but I'm going to move on from here.

Here is a gif demonstrating it (there are some artifacts due to compression),

{{< image src="/assets/img/crack_propogation_demo.gif" position="center" style="border-width: 1px; width: 500px;" >}}

Thank you for reading!
