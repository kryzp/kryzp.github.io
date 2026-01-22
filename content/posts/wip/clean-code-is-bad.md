---
title: "Clean code is (usually) bad code (in graphics, anyway)"
date: 2025-03-27
draft: true
---

Today, a video by Casey Muratori (who is an absolute genius by the way, the Handmade hero series is an absolute gold mine) titled ["Clean" Code, Horrible Performance](https://www.youtube.com/watch?v=tD5NrevFtbU) appeared in my recommended, and immediately it struck a chord with me when I remembered my experience with the entire field of computer graphics.

So, when I was first doing graphics, my thoughts were that I wanted to make it as general as possible. That way I could nicely support every API with a single exposed interface that you could call functions on and it would map to whatever Vulkan, DirectX 12, Metal, etc... backend was currently implemeneted, so I would only have to implement e.g: one particle system.

The problem with this approach is that it fundamentally misunderstood that graphics API's just aren't nicely compatible. Graphics programming is all about low-level optimisation, don't call any more functions than you need to.

Probably the key lesson to learn here is *don't prematurely try to abstract away code*. Write everything as messy as you want, then reflect on the code, identify key patterns being repeated, and extract them. The necessary abstraction layer that you need will reveal itself in time.

Essentially, if you were to write whatever code you need using whatever API you want, e.g: Vulkan, using no abstractions or anything, and made it as optimised as it possibly could be making only the exact specific API calls you need, then AFTER doing your abstraction layer at minimum you should still be making only those specific function calls. If you aren't, you've done something wrong. Clean code means completely nothing if code 'prettiness' is being placed above actual tangible performance.
