---
title: "Crack Propogation Simulation"
date: 2025-06-01
---

I recently watched [this](https://www.youtube.com/watch?v=0ebPkjqV7jE) video by Alexander Sannikov which was part of a larger Path of Exile rendering talk and found it pretty interesting. I figured that as a small weekend project I'll try to implement something similar because I'm bored.

The basic idea is that you create a triangular mesh of points, with the edges being ideal springs (ones which abide Hookes' Law) that slowly reduce their natural length over time until they hit a minimum natural length, to simulate drying. This is called a "quasistatic" simulation because it is assumed to happen so slowly that all inertial effects are negligible. To make sure all springs don't just all break at once, we slightly vary the spring constant and breaking tensions per spring to emulate microscopic impurities.

# Model Definition

Each point has a position $\mathbf{x}$, velocity $\mathbf{v}$, sum of forces acting upon it $\mathbf{F}$, mass $m$, and a toggle to determine if it's fixed or not.

Each spring has a natural length $L$, a spring constant $k$, a breaking strain $\varepsilon_T$, and a toggle to determine if it's dead.

Finally the simulation itself has a couple of parameters,

- Minimum spring length factor $\alpha$: Determines the smallest length of each spring as a fraction of its initial length.

- Velocity dampening factor $\mu$: How quickly each node loses velocity

- Shrinking rate $s$: How quickly the springs "dry out"

- Minimum and maximum range values for $k$ and $\varepsilon_T$ when randomly assigning them to springs: These small differences give rise to cracks, otherwise the whole simulation would break at once

# Creating the mesh

Since the mesh is perfectly regular we don't need to use any super complicated triangulation libraries or anything. Instead, I just create a regular rectanular mesh except each layer the direction of the diagonal edge gets flipped. Then every other layer just gets shifted half the `dx` to the right.

![](/assets/img/crack_propogation_mesh.png)

# Simulation

First, we go through all springs and apply their forces to the connected points. To do this, we calculate the length $\ell$ between the two connected points, and then calculate the strain as,

$$
\varepsilon = \frac{\ell - L}{L}
$$

If $\varepsilon \geq \varepsilon_T$, then we mark the spring as "dead" and exit there, this spring is no longer simulated. Otherwise, we calculate the force magnitude between the two points as,

$$
F = k \cdot (\ell - L)
$$

And then add the corresponding vectors to the force accumulator $\mathbf{F}$ for both points.

Then, we shrink the springs natural length to emulate drying,

$$
L_{n+1} = L_n - D(L_n) \cdot s \Delta{t}
$$

We have a function $D(l)$ which is meant to model the rate at which shrinking occurs. It must be zero when the length is equal to the minimum possible length of the spring, given by $\alpha L_0$.

$$
D(\alpha L_0) = 0
$$

For my simulation I went with a linear model:

$$
D(l) = l - \alpha L_0
$$

Though I'm curious as to what other forms of patterns might emerge if we were to use a different model. Food for thought.

Finally, we iterate through all points (that aren't fixed) and move them using semi-implicit Euler,

$$
\begin{aligned}
\mathbf{v}_{n+1} &= \mu \cdot ( \mathbf{v}_n + \mathbf{F}_n / m \cdot \Delta{t} ) \\
\mathbf{x}_{n+1} &= \mathbf{x}_n + \mathbf{v}_{n+1} \cdot \Delta{t} \\
\mathbf{F}_{n+1} &= \mathbf{0}
\end{aligned}
$$

Notice that since this is a quasistatic simulation we don't account for inertial effects (we assume acceleration is negligible) so acceleration is never directly kept track of.

# Summary

I wrote it in C++ using SFML and the whole thing fits into maybe 2 or 3 screens of code. There's plenty more that could be done (rendering is the biggest bottleneck) but I'm going to move on from here.

Here is a gif demonstrating it (there are some artifacts due to compression),

![](/assets/img/crack_propogation_demo.gif)

Thank you for reading!
