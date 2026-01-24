---
title: "A Brief Tour of Game Engines"
date: 2026-01-23
draft: true
---

A while ago I gave a talk on game engines at my universities game development society, and I figure it would probably make a good post too so here we are.

I'm by no means an expert on game engines or anything this was just a casual talk so apologies if there are any errors.

# What are Game Engines?

Initially, you'd probably associate game engines with "that" UI, like you'd see in Unity or Godot.

The reality is that game engines have a far broader definition, and are really just any shared code and features between games.

> Wikipedia: A software framework primarily designed for video game development, including specialized software libraries, and tools such as level editors.

Game engines are in my opinion some of the coolest pieces of tech we have, as they encompass such a wide variety of different programming problems. Adn this shows, as they're probably up there as some of the most complicated pieces of software on the planet, with AAA engines easily hitting millions of lines of code.

Unity, Godot, Unreal and GameMaker (the "big four") encompass a very small subset of what they are.

# Two Categories of Game Engine

Game engines can be generally split into two categories,

- Commercial: Often licensed out as part of the developers core business model, think Godot, Unity, ...

- Proprietary: Custom in-house engines made by studios to streamline game development, think Source (Valve - Half Life, Portal, Counter-Strike, TF2, ...), Decima (Guerilla Games - Killzone, Horizon), RAGE (Rockstar - GTA, RDR) and Frostbite (EA - Battlefield). However indies use game engines too!

## Commercial Game Engines

The terminology actually kinda sucks because it implies that the product has to be sold / licensed out for money.

Godot counts as a commercial game engine despite being completely MIT free licensed.

In reality, a commercial engine is any engine that can be used to make a commercial game. Which also means that proprietary engines are also commercial engines but typically this term also implies that it's an engine made by a company as the main product.

- Unity makes a game engine, licenses it out as a product
- Valve makes a game engine, makes games as the product

Commercial game engines must support all possible kinds of game, on all possible kinds of platforms, which is literally a herculean task.

As a result, they become super bloated, super fast. Even so-called "lightweight" game engines like Godot are massive.

## AAA Game Engines

AAA studios have an advantage as they don't need their engine to support all kinds of game - only what the company needs.

Some studios even have the advantage of knowing exactly what platform specs they'll be solely publishing on, such as console-exclusives (Naughty Dog, Guerilla Games, ...).

This makes comparison between proprietary engines and commercial engines stupid (in my opinion!) because of course AAA engines are going to be ahead in their tech compared to commercial engines because they're focusing on a subset of the features commercial engines have to support.

However, AAA engines tend to be more fragmented (and as a result have a very different workflow), as a result of their development history.

They typically have seperate tooling for building levels, editing entities, managing the asset database, etc... (think the Hammer level editor for Source Engine).

For the programmers, the engine is literally just a massive, bloated, spaghettifies C++ library.

- Frostbite is the most egregious example of this, being notoriously difficult to develop anything with due to over-complexity.

This is in stark difference compared to Godot, where everything is neatly integrated under one application that "magically" links everything together... somehow???

### Marketability

Your average gamer looks pretty much like this,

....

Nobody actually knows what a "game engine" is, and AAA companies know this, so they brand their engine as a "secret sauce" that elevates their games above the competition, making gamers think that your game is somehow a more "bespoke" product.

The extra marketability is great because game engines are incredibly difficult to maintain and develop.

.....

Gamers like to blame arbitrary "game engines" for terrible games, without taking into consideration what I said before.

Naughty Dog doesn't have to make a game engine that supports every game possible under the sun, but Epic Games does. You simply can't have your cake and eat it too.

Unreal Engine being more versatile comes with performance costs because Epic ultimately doesn't know what game it's going to be used for.

- Platformer? Top down? Side scroller? Open world? Will there be terrain? Will it be dynamically generated on the fly? Will you be doing live asset streaming or just load everything up-front? How complex is that AI? What's the graphics style and art direction? Will you be using more advanced volumetric effects or just billboarded particles? ...

### Engines do matter

I know this is a hypocritical statement considering what I said before, but the statement that "engines don't matter - design does" is really only true to an extent.

Engines heavily inform the design decisions behind games.

For instance, Minecraft *could* have probably been made in Unity if they tried hard enough, but it realistically wouldn't. The development would be going against Unity's architecture (think, custom physics, rendering pipeline overhaul, revamped networking, ...). And as a result, downstream effects such as the modding scene would look completely different, potentially even non-existent.

However, I still stand by saying that "Unreal is ruining games" is more like a half-truth.

## Indie Engines

The core piece of advice here is KISS - Keep It Simple Stupid!

Never, ever, ever for the love of god over abstract.

- Graphics code is already a massive PITA, especially when you're starting out, don't make it harder on yourself.
- Over-abstraction generations too much friction and code to maintain for small teams.
- This hinders the rapid development which makes indies stand out.

I'd recommend making a game alongisde your game, because it's easy to make a game engine that "works" on paper, but sucks in practice. There's no better way of finding bugs in your code and testing usability, and this will naturally shape the engine into your ideal tool.

# Typical* Engine Structure

Engines are typically organized into "tiers" or "layers"

- TOOLS FRAMEWORK
	- Level editor
	- Asset pipeline
- GAME CODE
	- Game-specific behaviour
- CORE
	- High-level rendering code (particles, materials, ...)
	- Physics
	- Entities
- PIGS (Platform-Independent Game Systems)
	- Graphics API abstraction layer
	- Input management
- OS
	- Registering the window
	- Event handling
	- Code varies depending on the platform.

.....

(Screenshot from Guerilla games talk on their tools pipeline)

I recommend watching / listening to [this](....) talk by Jason Gregory that talks about how Naughty Dog develops their games.

Their project architecture is organized into,

- Game Specific Code
- Common code (shared code except things like #if/#endif's are allowed for a few special cases per game)
- Shared code (doesn't change regardless of game)
- Low-level platform OS code (varies depending on OS)

Common code is then constantly abstracted to "promote" it to Shared code. Common code and Shared code both live in the same directory, but Shared code is compiled as a seperate library while common code is compiled as part of the game executable.

# A History Lesson

## 1990's: Pre-Engine Era

Around this time, game engines weren't really a thing, and every game was built from the ground up.

This era was pretty much dominated by id Software's pioneering work,

- Wolfenstein 3D (1992)
- Doom (1993)
- Quake (1996)
- id Tech 2 (1997): Beginnings of modern engine design

id Software later licensed out a variant of the Quake engine, "GoldSrc", to Valve, which they used to make Half-Life.

## 2000's: Standardization

Game engines start to become more standardized: level editors, asset pipelines, software libraries, etc..., all become commonplace. Multiplatform game engines become more prevalant due to the increased profits that they can bring.

- Licensing out your game engine out to others becomes a valid business model due to the gaming marked boom.

## 2010's: Commercial Engines take off

So now commercial and proprietary engines alike have established their toolchains. Commercial engines become very popular due to the indie boom where pretty much everyone and their mother is making a video game. Amidst all of this Unity becomes the de-facto standard indie engine.

Games - and by proxy the engines - focus on open world spectacle. As a result game engines start to become more generalized due to the increased demands in game scope, moving away from their specialized designs. A kind of "great filter" occurs where lots of engines are dropped around this time because their codebase simply can't keep up.

### Pipeline Modernization

Initially, tools such as the asset manager and level editor were seperate programs due to necessity. Lower hardware specs meant that level designers couldn't handle having the game, assets, level editor, and whatever else open at all times.

However, this was a pain. Multiple different programs created lots of friction slowing down development time and causing lots of bugs. As hardware specs improved across the board, this wasn't the case anymore. Modern level editors, such as Hammer 2 (Source 2) are literally built in the engine itself.

This allows level designers to hot-modify the game as it simulates and renders and get real-time feedback, and artists to see what their work looks like directly in-game instantly.

### Tools Pipelines

Many AAA companies have established pipelines dating back to the 2000's when engines were first standardized, because artists, level designers, programmers, etc... all have to work together... somehow(?).

Changing the established pipeline is incredibly difficult even if the current one sucks. If even a single feature is missing then people will just go back to using the old system.

[This] video by Guerilla Games talks about a lot of their difficulties updating their tools pipeline for the Horizon games.

As a result, most AAA companies are still stuck with at least parts of their pipeline fragmented between different tools. Oh, and don't even think about rebuilding the engine itself. Engines don't get "rebuilt". Ever. I think this image demonstrates that the best :p.

........

## 2020's: Modern Era

So now the focus becomes on photorealism and performance. Thanks to at least partially integrated tooling, workflows are far more streamlined.

We also start seeing more "convergence":

- Modern games need to do more and more each year

- They require more generalized game engines

- All but the largest studios are turning to solutions like UE5 because of the exponential costs of maintaining a competitive engine alongside building quality AAA games. Controversially, CD Projekt Red switched to UE5 from their in-house engine for the next entry into the Witcher series.

# Conclusion

In my opinion, indies actually have a leg up. Remember, your engine doesn't need to do *everything*, only what your game needs. And that is actually very manageable.

You reap performance benefits, and it allows you to do fancy rendering tricks that might be out of reach in commercial generic engines. It also gives you complete authority over the physics engine, entities, input processing, etc...

Celeste and Teardown were both built in engines becasue the developers felt that the extra control was necessary.

Finally, you do get those fabled bragging rights and marketability, and if you really need me to tell you, it's a great thing to put on your CV.
