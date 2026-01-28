---
title: "First Hackathon"
date: 2025-03-24
---

So last weekend I had the wonderful opportunity to participate in SotonHack 2025 with some of my friends, a 24-hour long hackathon hosted by my university's Electronics and Computer Science department. We got to pick between two themes,

1. Build an app that would solve a problem posed to people in the olden days (could be the 1600's, Rome, Ancient Greece, Victorian England, etc...).

2. Build an app that helps "shape a securer tomorrow" (the second theme was being sponsored by a seperate but closely intertwined society focusing on cyber security).

We eventually settled on the second theme, and built an app focused on tackling the risk of 'shouldering' - when unauthorised individuals can 'look over your shouler' where you can't see them and watch you e.g: input your pin code. It could also be used as a backup layer of security, like if you left your computer unlocked by accident when you left to go pick up your coffee at a cafe.

To cut to the chase: we won second place.

# How did it work

- We maintain an internal whitelist of 'authorized' face encodings which can be assigned a name and registered / removed at will.

- The app then perpetually runs in the background, periodically checking the webcam for all the faces that it can find.

- Whenever it detects an unauthorized face, it 'locks' - immediately covering the screen and blocking all input until only authorised faces are visible.

- After detecting someone unauthorised it saves an image of what it's currently seeing every 10 seconds, and every 2 minutes it sends an email containing that image to whomever the device belongs to.

- Finally as a failsafe, there is a secret key combination can force you out of it, and it isn't `alt-f4`.

We used Python for it with some facial recognition libraries and Tkinter for the UI. The performance was laughably slow ("12 frames per year") due to some poorly written multithreading, but whatever it's a proof of concept.

# The whole story

That wasn't our first idea. **IF** I'm being honest it was kind of a disaster in the first half. We were initially going to do the first topic, and were stuck between three ideas,

1. Napoleonic-era battle simulator, you're given a fixed amount of cavalry / artillery which can move at a certain speed and you need to beat increasingly bigger armies controlled by AI.

2. Victorian-era trade optimiser: Given some amount of countries with a certain amount of resources (positive is a surplus, negative is a defecit), what is the optimal trade that each country should be doing (e.g: trading 1.5 wood and 0.1 steel for 0.9 coal) for each other country to minimise their defecit using their surplus.

3. Cold-war-era Soviet space program game where you're given some amount of money / resources with which you can get some amount of parts (each of which can add to the mass, fuel, fuel-consumption, drag, life, stability, etc... of the ship, along with a required astronaut cabin) that you can stick together like lego bricks. Your goal is to get as high as possible to be able to brag about the fact you did so and beat the Americans.

All of these sound great, but unfortunately they were really abstract and difficult to split between us to work on. Of course, none of us knew this so we blindly continued on. Oh, and I thought it would be a great idea to use Java with a library nobody else had ever used (libGDX) for the graphics! What could possibly go wrong?

We decided on doing the Victorian trade optimiser since it seemed the most computer-science-y, and it was one where everyone felt they had an idea of what they could work on and such.

# Problems

Literally everything went wrong.

I was working fine on the project since I knew what the library was, and wrote up a basic main menu, but nobody else knew how to use it. We were kinda just walking on ice, if that makes any sense.

My role was the backend algorithm that actually found the maximum possible trade. My idea was to use game theory. For $n$ resources there's going to be $\mathcal{O}(n^2)$ possible trades that could occur between a buyer and a seller, so what's the best possible strategy for the buyer to maximise their earnings?

That sounds like it would work, but the problem I didn't forsee for some reason is that not only is this incredibly hard to do for an abstract number of resources, we also aren't accounting for all the *other sellers* that the buyers being sold to is actively also trading with, which might influence how it percieves the value of each resource it's buying, which influences what the optimal trade it. It felt like I was both under and over-complicating things. I just couldn't get it to work, and sat at the whiteboard with my mind blank.

I'll spare you the details but at around 10pm we ultimately just decided to move away from the Victorian trade optimiser since I couldn't get the algorithm to work, which was for the better anyway since more of us were familiar with Python, TKinter, and facial recognition libraries.

However, I was still super annoyed I couldn't get it to work. It felt like I was just missing one piece of the puzzle and it would all fall into place. Well, eventually out of sheer frustration, I managed to get the Victorian trade optimiser algorithm working using Monte Carlo sampling. [Here's how it works](/posts/victorian-trade-algorithm/).

Of course, by then it was too late and we had fully pivoted to the privacy app instead, which was for the better. I really shouldn't have been so insistent and I take most of the blame for us being so off track. That being said, I'm really damn happy that I got the algorithm working eventually haha.

# Fin.

The other projects showcased were also really, really great (and funny).

My particular favourite (and I believe what won first place in theme 1.) is a group of people that wrote an equivalent of the SEAtS app (an app the university uses to track attendance to lectures by having you write a unique code generated per lecture as proof of attendance, easily bypassed though since someone can just write the code in a groupchat).

It used AI-generated medieval paintings to encode a hidden unique code that you could scan to 'check-in'. It even ranked everyone on a global leaderboard with titles depending on their attendance (if you had >90% attendance you'd become a noble, and if you had <10% you would literally just be pronounced dead).

Overall though, even if it was pretty rocky, I'm really glad we did it. While it was super intensive and stressful, it was also really nice to just sit down, talk with friends, and hone my teamwork skills. The programming especially was very different from what I'm used to working on solo projects.

Excited for the next one already :)
