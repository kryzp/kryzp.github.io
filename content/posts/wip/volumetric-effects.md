---
title: "Volumetric Effects"
date: 2025-05-07
draft: true
---

Volumetric effects are (in my opinion) some of the most fun things to implement in graphics programming. There is something deeply satisfying about it.

The general equation is something like follows, where you integrate the total amount of light coming into the eye from the smoke.

$$
L_o = \int_{x_\text{eye}}^{x_\text{incident}}{e_\text{smoke} + c_\text{smoke} \cdot T_{E \to x} \cdot \rho \left( x \right) \cdot \sum_{L \in \text{lights}}{\left[ \int_x^{x_L}{c_L \cdot T_{x \to y} \ \mathrm{d} y} \right] } \ \mathrm{d} x}
$$

Where $T_{ab}$ is the transmission between points $a$ and $b$. That is,

$$
T_{a \to b} = \exp \left( -\int_a^b{\sigma_e \left( x \right) \ \mathrm{d} x} \right)
$$
