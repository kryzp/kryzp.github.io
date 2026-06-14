---
title: "Simplifying Vulkan Synchronisation"
date: 2026-04-02
draft: true
---

This is just a quick post to show you that synchronisation in Vulkan doesn't have to be a major pain in the backside, with a little thing called (drumroll...) Timeline Semaphores!

Note that unfortunately you will still have to have at least two "regular" semaphores for the core swapchain synchronisation - but those can be abstracted away into the backend. Other than those, you can create a simple wrapper around timeline semaphores and use those instead, making your code way nicer and simpler.

The basic idea is that we introduce an abstraction over a point in time - a "TimelinePoint" (yes very creative I know) - and then whenever you call to the backend to submit data to any given queue, those functions return a timeline point set in the future. After submission, we can continue doing whatever work we need to do, and once we need to make use of whatever resource is being worked on by the GPU (i.e.: wait for the GPU to finish), we can call to the backend device, asking it to stall until the timeline point it returned.

It should be easier to explain what I mean if I just give you the code directly, so here is the code I've taken from my Vulkan renderer, [Magpie](https://github.com/kryzp/magpie), which is written in C.
