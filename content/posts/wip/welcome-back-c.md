---
title: "Welcome back, C!"
date: 2025-08-13
draft: true
---

So, I rewrote my entire Vulkan renderer, *Magpie*, in C.

To explain why, I'm going to have to go way back to around 6 years ago now (whoa).

### GCSE

I had just finished Year 10 and was entering Year 11 of my GCSE's and I had come across "Handmade Hero" (just interrupting now, *please*, go and watch at least the first ~30 or so episodes, it really may be the single greatest piece of free game/engine/programming in general advice on the entire internet, no exceptions).

I found it fascinating and at the time I was still pretty bad at programming and Casey certainly looked like someone who clearly knew what he was doing so I started following along the series starting from day one.

Up until this point I had pretty much exclusively practiced OOP concepts since that was what pretty much all game development tutorials I found online showcased, and engines like Unity and Gamemaker especially re-enforced this idea that a game object can be directly translated into an object in the code. Intuitively it makes sense to have a "Player" that has a position and can "move()" or "shoot()".

Casey's ideas completely contradicted this, in a good way. It was super fascinating to see how he approached problems and how much simpler and clearer his solutions were to what I would have done.

Anyway, after a couple episodes I considered myself "good enough" and tried making a really simple C project, *this* ascii game. This backfired because I never actually understood the core principles of what Casey Muratori was trying to teach about programming and basically all I took from it was just "structs and functions are seperate", but in the wrong way.

Instead of actually doing proper procedural programming and asking myself "what information do I have and what functions do I need to operate on this state in order to transform it into the state I need", I was basically still thinking "okay I have a Player class except now I have to write ``Player_Update(&player)`` instead of ``player.Update()``. Oops.

Of course, this mentality completely failed me. I basically tried to implement things in a regular OOP style, except now with both my hands tied behind my back because I was suffering from all the disadvantages of OOP without any of the benefits of a language that actively supported it. So naturally young me thought this was all a load of crap and moved on, and continued to regularly practice OOP in Java and C++ alike.

### A-Level

Fast forward a couple years, I had finished my GCSE's, had moved on to Sixth Form and for my Year 13 A-Level Computer "Science" NEA (non-exam assessment) we got to write and program we wanted as long as it satisfied the need of our "client" (who we could make up) and wrote about what we made and why we did it.

I decided to write a high-performance VULKAN GAME ENGINE IN C++. Note that I had *never* touched Vulkan *ever*. Naturally, this seemed like a great time to apply all my beautiful OOP knowledge and put it to the test.

The engine was (still is) a complete mess.

I still scored high on my NEA because it's not really about your code "quality", but whether or not your code satisfies your clients requirements properly, and how well you can yap about a single function call in the write-up you do. Probably helped that I wrote so much code that I printed out maybe 500 sheets of paper in total for it so it's not like it could have been properly graded anyway, :p.

But even as I was writing my code I knew that it was fundamentally terrible.

I, in my infinate wisdom, not only tried writing a Vulkan abstraction layer without ever having even seen what Vulkan is like beforehand, but also decided it's a great idea to wrap *everything* in an interface. And I mean *everything*... *EVERYTHING!!!!*.

This, as you can imagine, lead to a frankly unfathomable amount of friction. So much so that I started to subconsciously try to ignore actual improvements to the rendering architecture, and knowingly used super slow methods of rendering not because I didn't know any better, but because it was simply such a massive time commitment to expose even a single function to the high-level renderer.

I mean, lets just look at what I needed to do just to expose a single function to the renderer. First, I'd need to add the function to the graphics backend interface, which meant the function call couldn't contain any Vulkan types because it had to be generic (yay!!), then I'd have to copy that function call to the vulkan backend header file AND source file. Only to then find that the function actually required some extra information from the renderer so time to modify the same function in three files again, one of which is in a completely seperate directory.

If that function needs, e.g: Vulkan specific enums I'd need to create my own version of those, and then create a translation function that converts the engine-level enums into the equivalent Vulkan ones, and back. Better hope you don't make a mistake here!

And, of course, I was constantly paranoid about whether or not there was another way of doing things or if the functions I was exposing had parallels in DX12 or OpenGL, "just incase" I ever added another rendering backend. Yeah.

Plus, since I was only really familiar with OpenGL up until that point, I naturally wrote the abstraction layer in an OpenGL style. For those of you who don't know, OpenGL is very much not like Vulkan. Vulkan is more comparable to DX12 than OpenGL, and *really* doesn't lend itself nicely to the global state OpenGL uses.

I, of course, didn't know that so the graphics backend essentially maintained a constant global state that could be modified at any point and would automatically then hash these together whenever it needed the relevant/corresponding pipeline. If it didn't already exist in the internal cache, it would generate a new one. I used a similar method for binding textures and buffers because I didn't understand descriptors either (though I still believe descriptors are ultimately one of the big failures of Vulkan - needlessly complicated for what they are, hence why bindless is so popular). Command buffers were completely abstracted away which lead to a whole host of other problems too.

And you still need to integrate the rendering with the resource system for models and the entity system to have objects to render. All of those need their own handles which resolve in their respective systems.

Then, the custom maths and container libraries meant that not only could it be that the code was failing at the system level (e.g: entities not properly occupying their freelist) but the custom ``Vector``, ``Deque``, ``HashMap``, etc... implementations could also just be failing silently, and your shader buffers could actually be misaligned or improperly reading matrices because the layout of the data (or functions, e.g: creating a projection matrix) could also be completely wrong.

Oh, and dynamic allocation *fucking everywhere*.

Needless to say, I wouldn't say it's a stretch to literally say every single component of the entire engine, potentially every line of code executed, could be treated as a failure point. I hope that covers just how *dire* the situation was.

What's funny is despite all this crazy architecture the rendering code really did nothing complicated. It rendered a list of models with shadows. That was it.

Frankly, the whole project was such a pain to work on that it burned me out for months. Even now when I look at the code I get a weird uneasy feeling, it's hard to explain.

### University Year 1

So, continue to fast forward to university, in the beginning of my first year I decided that I wanted to take one more shot at writing a proper Vulkan renderer. By this point I was kind of becoming dissilusioned with OOP as a whole (and I'd say becoming an "alright" programmer) but I still stuck to it because it's what I was used to.

I stripped away all the garbage fluff from my NEA: entities, physics, custom maths and containers, networking, the plugin abstraction layer, etc... and only kept the core functionality I'd need to work with Vulkan, since I knew by this point I wanted to go into graphics programming in the future. I used the standard library for container types and glm for maths.

Now that I had this leaner, slimmer codebase without interfaces and abstraction everywhere, I could properly start to focus on actually learning how Vulkan worked, what it was trying to achieve, where it failed as an API (looking at you, over-granuality and the ≈ FIVE-HUNDRED fragmented extensions), and how to use it efficiently.

I started to actually abstract the things I needed, and only weakly, just enough to overcome the insane amount of boilerplate code Vulkan uses so my code could be readable. Naturally through experimentation I discovered that gpu buffers, pipelines and command buffers should be first class objects, rather than being abstracted away.

I understood what the point of command buffers even was in the first place, and they weren't just something magical I had to pass into certain Vulkan functions for some reason anymore.

Vulkan was so different from OpenGL and once I started taking away all the abstraction I needlessly created for myself originally I could start to see how I was *actually* meant to use it.

I actually made some progress, I had a (basic) render graph, model loading, physically based shading, reflections, HDR, deferred lighting, the list goes on. The *less* abstraction there was, the *easier* writing rendering code was. I could actually focus on rendering the objects I needed.

However, eventually found myself falling into the same pitfalls that I originally intended to get out of. I kept second guessing myself when it came to, for instance, where to place data.

et's say I'm writing a deferred renderer, where do I put my sphere mesh for the point lighting? The obvious answer might be to just dump it into the renderer class itself, but is that really where it belongs? Maybe it should go into a mesh manager class as a static member. But then what if I want different levels of detail for the spheres? Should I have a seperate lighting pass containing the sphere mesh? But then if I need the sphere mesh elsewhere I don't want to make another sphere mesh.

The data could go into any one of these places, but I need to make that decision now, without knowing anything about the future.

This wouldn't be so much of a problem if it wasn't such a pain to refactor code later down the line. OOP is notorious for a) constantly needing to be refactored yet also b) being incredibly slow and tedious to refactor.

Especially for something like C++, where the refactoring tools are nowhere near as good as ones for e.g: Java. So naturally to try to prevent refactoring (or rather, to minimise it as much as possible), im encouraged to just sit there evaluating where I should put a fucking sphere mesh(???).

These seem like such useless questions, but it genuinely started to bother me heavily. Just like in my old A-level game engine I found myself passively resisting any form of complex features to add because of minute decisions like these that really shouldn't even be a question in the first place. I should be trying to get some code to run in a certain order and instead I'm over here worrying about what goes where like some librarian.

Anyway I dropped the project for a month or so because I'd become annoyed with opening it, and when I did decide to work on it I barely made progress and just blankly stared at my screen because it felt like anything I would write would just be a mistake.

### University Year 2

That leads us to now, a little more than a month away from the beginning of second year.

Eventually I decided to start programming again but this time I decided to revisit Handmade Hero just because it popped up in my feed, maybe I'd missed something crucial and it's not like I was doing much over summer anyway.

I don't know why, but for whatever reason things just "clicked" for me this time around. Like I wasn't looking at the code as treating structures like objects, but simply just *operating* on the structures. The data is just that, data. And fundamentally the idea of attaching functionality *to* that data itself was stupid, the program just acts *on* the data. It gets *transformed*. It was genuinely eye opening.

The more I listened to him rant about certain things the more I started to question why we did things the way we did. Getters and setters, private variables, inheritance, virtual functions, everything. The more I thought about it so much of it seemed like a waste of time that just took from the clarity of the problem to be solved.

For whatever reason, his video on introspection (I didn't watch all the way up it, I just skipped to it) was what really made me understand. The "tokenizer" for instance, is just state. The functions just operate on the state given to it, and maybe update it, but the tokenizer itself isn't something with functionality, it's just state.

Memory arenas were another thing that once I learned about them I started asking myself how I hadn't been taught something so simple. I mean, I'm entering the second year of University and they haven't taught us basic shit like this? What? I reccommend Ryan Fleury's talk, ["Enter the Arena"](https://www.rfleury.com/p/enter-the-arena-talk).

Memory arenas solve such a core problem of memory management and make dynamic allocation almost free while also reducing the number of failure points of your entire program down dramatically. As Ryan puts it, they change memory allocation into a question of scope rather than a question of ownership, which is not only far easier to manage in the code but also far easier to reason around.

You can just "declare" certain arenas to be never reset throughout the entire program, while others might reset per frame, and others are used for as a "scratch" for intermediate calculations. Crazy cool!

So feeling *actually* confident and knowledgable this time around, I decided to try do something insane: rewrite my entire Magpie renderer in C.

So I did! You can find it on my GitHub and in my humble opinion it pretty damn good. Not only that, but adding new features to the renderer is a piece of cake, and I can't wait to start experimenting more.

I think my core take away from all of this is that object oriented programming, *FUNDAMENTALLY*, doesn't lend itself to rendering. It simply doesn't. It masquerades itself as fitting it, types like "image", shader" or "gpu buffer" really seem like they lend themselves nicely, but they *don't*. They're just information that can be *acted upon* by functions to *transform*, be it transitioning the layout of the image, or copying memory between gpu buffers.

Oh, and yeah. It's been a while. I've got a ton of WIP posts, but they're only done when they're done.
