---
title: "Bikeshedding, Scope Creep and Gamedev YouTube"
date: 2025-03-30
draft: true
---

> Bikeshedding: The futile expenditure of time and energy in discussion of marginal technical issues while avoiding more complex or important issues.

Everyone is guilty of this, and we see it everywhere.

Note taking is infamous for this, nowadays we've got all these fancy note editors like Notion, Obsidian, or whatever new slow bleeding-edge AI predictive software is the new rage right now, that don't fundamentally solve the problem (at least for me) because people convince themselves that the reason they don't have great ideas or productivity they dream of is because they can't visualise or interconnect them in the same way Notion or Obsidian can.

A much harder pill to swallow is that an empty text document or a physical notepad is pretty much as good as it gets. This extends to coding too, with people insisting that visual studio has nothing on the superiority of the TetraDeath 9000 text editor that has been community developed since 1826 by one person part time and just recently implemented undo functionality!

The best example I think of is the (modern) gamedev YouTube scene.

### Game Development YouTube

Gamedev YouTube *used* to be about making a log diary of your process as you worked on your game. This was awesome as it meant you'd advertise and build up a following for your game before it released, you'd be less likely to drop it due to the community support, you could even open up a patreon and (if you got lucky) you'd make enough money to go full time.

ThinMatrix comes to mind, and to this day is probably the best game developer making consistent down-to-earth development diary logs (devlogs!). He's the reason I got into graphics programming in the first place, my first ever renderer was the one following his LWJGL2 + OpenGL tutorial, but in C++. I owe him and his channel *a lot*.

Around 2020 there was a massive boom in the amount of people watching gamedev YouTube, so naturally more people started making dev diaries. However, the demographic of people watching shifted as well, and it wasn't just actual developers watching each others work, it was regular people.

As a result, the devlog style shifted to accomodate the new demographic. ~~Developers~~ Devtubers focused less on the actual gamedev part (talking about the tricky programming challenges, interesting solutions, how they iterated on the artstyle, etc...) and more on an idealised, over-edited, over-simplified form that had mass-appeal. This continued to morph and evolve until it stopped being about having a devlog diary to promote your game and your passion, and more about making "a" game so that you could have something to make content about to push out the door.

There's nothing inherently wrong about that - some people just preferred making YouTube videos and game development was fun. That's great! The problem was that these people fell in the same bucket as the "career" developers who did YouTube on the side, and so they had the same "credibility", if that makes any sense.

Something that frustrated me to no end however was seeing the same people start making tutorials on game development even though they clearly didn't know how to properly code or paint or compose themselves. The code was sloppy, slow and intextensible. The art was uninspired, rushed, barely respected pixel boundaries ("mixels") and abused banding. The music would be passionless repeating 8-bit ear-grating melodies without any insteresting counter-melodies or varying instruments or change of key or anything.

I would know this because at the time I myself was learning game dev, and I would follow these tutorials. AND I would be then pulling my hair out over bugs I didn't understand, or I would try to modify the inventory tutorial I followed only to find it was built around specifically a certain type of item with trivialised properties like "damage" or "healing". My music wasn't anything like I imagined and I couldn't get any of my ideas into good looking pixel art.

I'm not saying there's anything outright wrong with breaking rules, but it must be done intentionally. Mixels *can* look good **IF** you know what you're doing (but banding pretty much universally looks bad / worse), and you *can* get away with sloppy code **IF** it isn't being run hundreds of times per frame.

I guess another deceptive part of retro art is that due to its nature it seems easier than regular art. Turns out it's about the same, if not harder in some areas. Any minute change to a pixel can completely change the way you percieve the depth and shape of the artwork. Yet, the same fundamentals about shading, primitives, and colour all still apply.

The reality of the situation unfortunately is that you *can't* "just copy" from tutorials. They're teaching the wrong thing. What they should be teaching is just basics of programming because implementing a Diablo-style inventory is far more difficult compared to a list you can just scroll through.

But this fundamentally contradicts with the people watching these videos, who want immediate results whether they want to admit it or not. The idea of having to learn basics of programming logic, or art, or music, or anything before implementing a cool inventory system or a sick bullet hell shooter is incredibly discouraging.

### Anyway...

Here's where we come full circle: Bikeshedding and Scope Creep. Modern DevTube relies on a constant flow of progress being made on your game to have videos to make about.

However, games are difficult to work on. Implementing core game systems is harder and adding new content is even more difficult, not to mention 50% of your ideas will probably get shelved while the other 50% will get re-worked multiple times before they're deemed close to finished.

This can be compounded by the fact a lot of people want the bragging rights of saying they "made their own in house engine" and become convinced that a custom engine is the only way to create the "game of their dreams", yet when you ask them for the specifics of the game they're imagining, they can only give vague ideas like "slick combat" and "unique exploration" (????).

Doing constant reworks becomes a quick way to "work on your game" while having something to make a video on, but you're ultimately running on ice. The game isn't actually being worked on. If you don't know what your game requires, why are you adding those features into your engine?

Usually it's because people don't want to confront the fact they don't know what the game they're making actually is. And that's okay! Game development is hard, but you need to face that beast before it overwhelms you and your project, and you find yourself 5 years in with a game that looks less finished than the prototype you had working in the first week.

It seems like an obvious fact, but you shouldn't invest a ton of time into something that,

1. You don't know if it would work or look good in practice.
2. Even if it would, would it result in too much work to get functioning regardless? Work that could be put into core gameplay features?

The key word here is in *practice*. Plenty of things sound amazing in your head, revolutionary even, but they just don't make for a fun experience. Let me be the first to tell you that there is a reason most games have a simplified inventory system that lets you carry around 10 guns, plenty of ammunition, a rocket launcher and enough food to feed a small town, while letting you take two shots to the face for just half your imaginary healthbar which a bandage around the arm fixes lickety-split (looking at you, Bethesda).

The games that do otherwise do so intentionally, not just "because" it's different.

...Ultimately something many people have to understand about game code is that at some point it will just become a mess. There's not really anything you can do, there's no one-size-fits-all entity architecture, editor, renderer, physics engine, etc... And that's kind of the point.
