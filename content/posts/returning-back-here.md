---
title: "Returning back here"
date: 2026-01-23
---

It's been a while.

I've been busy with tests (which are now done, thankfully) and generally lost passion but I'm trying to get myself to commit to this better this year.

As part of this, I've remade the website using Hugo as I thought the system Hugo has, including the themes, is just better compared to Jekyll. I just kinda wanted a fresh start. Later I also want to go back on the previous posts I've made (of which there isn't a lot) and redraw some of the diagrams, since I think they're a bit sloppy.

In terms of programming, I've been back to work on my Vulkan renderer, [magpie](https://github.com/kryzp/magpie). I've actually remade the thing in C++, though the core project structure is actually still very C-like.

I've got a pretty good render "graph" (though currently more like a render list) going, and I've been focusing my efforts on getting to feature-parity with the old version. Additionally, it's got some other nifty features, like a slightly nicer asset system and a multi-threaded fiber-based job system!

Ok here's a quick todo list for what I've got next:

1. Achieve feature parity with old engine, so basically a basic PBR indirect renderer with compute culling.

2. Improve the Render Graph with a builder pattern

3. Check if I have to use a seperate thread for polling SDL or if I can just convert the main thread to a fiber directly :p. Maybe a special flag for jobs that have to stay on the same thread?

4. Refactor the engine to use a plugin system for all components like in TheMachinery. Initially just bother with the platform layer because other modules are still w.i.p. and I don't want to over-abstract.

5. Switch to using timeline semaphores over fences for synchonization

6. Add some more ImGui tooling, especially a profiler for both the CPU and the GPU.

7. Refactor code, move rendering code out of App and into a RenderManager of some sorts.

8. Improve the top-level material and scene system.

9. Start going through the ideas / todo list.

Oh, and I'd like to find an internship this year... soooooo...
