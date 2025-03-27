---
layout: page
title: Projects
permalink: /projects/
---

This is a sort-of complete list of things I'd label as projects. The more "notable" things are also on my CV.

### [Lilythorn](https://github.com/kryzp/lilythorn)
A physically-based rendering engine implemented via Vulkan and SDL3, using a hybrid forward/deferred clustered rendering system. Integrated Assimp for model loading, in addition to general resource management of textures, shaders, buffers, etc... which all ties into the bindless material system. Has support for both point lights and directional lights with optional shadowmapping. Built a custom Vulkan abstraction layer to simplify resource handling, in addition to custom container types. Also designed with cross-platform compatibility using MoltenVK on macOS in mind. Additionally, has other more specific features, such as GPGPU-driven particle systems, volumetrics, and realistic water based on the fourier series of real ocean waves.

### [Leviathan](https://github.com/kryzp/leviathan)
A framework for creating 2D games inspired by FNA/MonoGame, written in C++ using the SDL and OpenGL libraries. It features its own custom maths library and container implementations, and handles basic asset loading/unloading, entity management, input and event handling, and rendering.

### [Path Tracer](https://github.com/kryzp/pathtracer)
Simple path tracer based on the raytracing in one weekend series, it traces multiple rays from the cameras view source position accounting for surface properties, which it then uses to create a final output image.

### [2D Finite-Element Solver](https://github.com/kryzp/simple-finite-element-solver)
Given a set of 2D beams of different materials and shapes inter-connected between each other, a set of input forces and boundary-conditions, it forms a stiffness matrix which it uses to form a linear equation that is then be solved for the deflection of each point.

### [2D Navier-Stokes Fluid Solver](https://github.com/kryzp/simple-navier-fluid-sim)
Iteratively updates a fluid velocity-grid following Navier-Stokes equations by first applying external forces, then applies the diffusive term, and finally applies advection. In-between, it re-projects the velocities to keep the divergence of the field zero.

### [Wyvern](https://github.com/kryzp/wyvern)
I wrote this game engine for my Computer Science NEA. I'm glad I did it, and its probably the only reason I ever had the courage to really jump into Vulkan, which was super intimidating at the time. That being said, the code hasn't exactly aged very well. It's got incomplete physics, a barebones entity system, the input system is still pretty good (does what it needs to do), and the renderer is a complete mess. I was coming from an OpenGL perspective so I tried to mold it into... that.

The renderer maintains a global state and builds pipelines on the fly (which isn't a bad thing but maintaining global state like that isn't exactly great). It was incredibly messy to use and shader buffers working was what could only be called a miracle. It was inextensible, and finding bugs was a nightmare (didn't help I also made my own maths library and couldn't figure out how to get the data to align properly, so at any point I had a rendering issue it could have equally come from ten seperate different things). Good times.

When I started [Lilythorn](https://github.com/kryzp/lilythorn), I used the rendering backend from this as a "base" to start from, though it has been pretty much completely re-written by this point.

### [Pathtracer](https://github.com/kryzp/pathtracer)
My first attempt at a simple pathtracer. Exports to PPM.

### [Advent of Code](https://github.com/kryzp/advent-of-code)
My solutions to Advent of Code! I have yet to do 2024 and 2025...

### [Project Euler](https://github.com/kryzp/project-euler)
My solutions to some Project Euler problems!

### [Monte Carlo Robot Localiser](https://github.com/kryzp/mcl-pos-vel-particle-filter)
Learning about Monte-Carlo localisation, I wanted to try it out on a really simple test case. I imagined a 2D robot on a plain grid, that moved at a constant velocity through space. The idea is the robot could only know a very broad idea of where it was - the grid coordinate. It's job would be to use this technique to try to figure out it's *exact* position and velocity. Imagine you have a map, and know you move from K8 to K9 to J9 to J10, what is your velocity and position? It worked pretty well, but I would like to revisit this in the future and make it more optimised and efficient. One more thing to add to the backburner...

### [AsciiLand](https://github.com/kryzp/AsciiLand-C)
My first "real" C project, I wrote it around Year 10. It featured a simple "pure" entity-component-system, map loading, "texture" loading, a renderer, you could move around the map between different chunks, pick up and equip different items, etc... I haven't tried running it since but I have no doubt it's full of memory mismanagement errors.

### [Battle Boats](https://github.com/kryzp/battle-boats)
Hilariously over-engineered battle boats that I wrote in year 12. Nothing much else to say about that.

### [Mandelbrot Set Visualiser](https://github.com/kryzp/mandelbrot)
Multithreaded Mandelbrot set visualiser written in Java using Swing.

### [WinForms Tetris](https://github.com/kryzp/tetris-winforms/)
It's literally just Tetris.

### [Small projects that don't justify their own repos](https://github.com/kryzp/small-projects)
  - Conways Game of Life
  - 2048
  - A* Algorithm Visualised
  - Towers of Hanoi recursive solution
  - Minimum Spanning Tree via Prim's Algorithm
  - Colour image to ASCII converter
  - BrainFuck interpreter written in C
