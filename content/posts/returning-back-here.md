---
title: "Returning back here"
date: 2026-01-23
---

It's been a while.

I've been busy with tests (which are now done, thankfully) and generally lost passion but I'm trying to get myself to commit to this better this year.

As part of this, I've remade the website using Hugo as I thought the system Hugo has, including the themes, is just better compared to Jekyll. I just kinda wanted a fresh start.

In terms of programming, I've been back to work on my Vulkan renderer, [magpie](https://github.com/kryzp/magpie). I've actually remade the thing in C++, though the core project structure is actually still very C-like.

I've got a pretty good render "graph" (though currently more like a render list) going, and I've been focusing my efforts on getting to feature-parity with the old version.

Additionally, it's got some other nifty features, like a slightly nicer asset system and a multi-threaded fiber-based job system!

In terms of what I'd like to do next, I've got a very rough draft,

1. Achieve feature parity with old engine, so basically a basic PBR indirect renderer with compute culling.
2. Switch to using timeline semaphores over fences for synchonization
3. Add some more ImGui tooling, especially a profiler for both the CPU and the GPU.
4. Refactor code, move rendering code out of App and into a RenderManager of some sorts.
5. Improve the scene system.
6. Start going through the ideas / todo list.

Oh, and I'd like to find an internship this year...
