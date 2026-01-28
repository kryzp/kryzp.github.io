---
title: "Fixed Uniform Load Beam Derivation, circa. 2023"
date: 2025-03-25
---

(This is an old text written around 2023.)

So I've recently been reading up on shear-moment diagrams and cantilever beams due to it being somewhat related to my EPQ for school. To anyone not from the UK, it's basically a thing some schools make students to do where they can build an artefact (make something) or do a dissertation (write something) on whatever you want.

Anyway I was quite dissapointed to find that pretty much all of the derivations that I could find of uniformally distributed load along a beam fixed at both ends were fairly... bad, and didn't really care to explain where they were coming from. It also didn't help that all of the videos featuring it were filmed in 120p in about 12 frames per hour.

After spending an embarrassingly large chunk of my day banging my head against a blank sheet of paper, here's my derivation,

We have a uniformally distributed load $q$ along a beam of length $L$,

![](/assets/img/beam_derivation.png)

Our boundary conditions are,
1. Zero deflection at $x = 0$ and $x = L$, so $y(0) = y(L) = 0$
2. Zero derivative at $x = 0$ and $x = L$, so $y'(0) = y'(L) = 0$

We know the beam is in static equilibrium and so we have an equal resultant force on both ends of the beam (A and B), call it $R$,

$$
R + R = qL \implies R = \frac{1}{2} qL
$$

This means that our shear function can be written as so,

$$
V(x) = \frac{1}{2} qL - qx
$$

Since,

$$
M(x) = \int{V(x) \ \mathrm{d} x} = \int{\frac{1}{2} qL - qx \ \mathrm{d} x} = -\frac{q}{2} x \left( x - L \right) + C_1
$$

By DI method, $y'' = \frac{1}{EI} M(x)$ (which comes from a curvature approximation),

$$
\begin{aligned}
y' &= \frac{1}{EI} \int{M(x) \ \mathrm{d} x} \\
&= \frac{1}{EI} \left( -\frac{q}{6} x^3 + \frac{qL}{4} x^2 + C_1 x \right) + C_2 \ \text{but} \ C_2 = 0 \ \text{by boundary condition 1.} \\
y &= \frac{1}{EI} \left( -\frac{q}{24} x^4 + \frac{qL}{12} x^3 + \frac{C_1}{2} x^2 \right) + C_3 \ \text{but} \ C_3 = 0 \ \text{by boundary condition 2.}
\end{aligned}
$$

Now we can apply boundary condition 1. (again) and solve for $C_1$, where we find that:

$C_1 = -\frac{qL^2}{12}$

So finally we find,

$$
M(x) = -\frac{q}{2} \left( x \left( x - L \right) + \frac{L^2}{6} \right)
$$

This also has the benefit that now you know $y(x)$, so you can also calculate the deflection at each point in the beam.

Now you can proceed as normal, most likely calculating the maximum moment applied to the beam.
