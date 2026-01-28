---
title: "Demystifying the TBN matrix"
date: 2025-03-23
---

If you've ever done graphics programming, you've likely implemented some sort of normal mapping. You will then know that it relies on something known as the *TBN matrix* to magically convert your tangent space normal map textures (the desaturated blue ones) into world space normals (the cooler rainbow ones) to be used in lighting. It was something that seemed really abstract to me when I first learned about it.

I mean, I kind of understood it, it was a matrix that moved us between two coordinate systems that meant that we could take our generic tangent space normal maps, and attach them to whatever surface we wanted.

That being said, I'd be lying if that meant I actually had a proper picture for what it was.

If you know a little linear algebra you'll know that an $n$-dimensional matrix can be interpreted as a collection of $n$ column vectors which lay out something known as a "basis". It's a way of writing what *their* coordinate system looks like, if you looked at it from the perspective of *your* coordinate system.

I.e: If I move one unit along the $x$-axis in their coordinate system, what would that look like in my coordinate system?

Similarly, here we are creating a basis describing what the local "tangent space" looks like at the fragment that we are shading, if viewed from the perspective of world space. How much is one unit moving upwards in tangent space, in world space?

And that's really the way I think people should look at this matrix, it's a mathematical 'rosetta stone'. It "translates" the alien tangent space language into our readable world space language. Of course, tangent space is pretty understandable, so I wouldn't go as far as to call it 'alien', but you get my point.

![](/assets/img/tbn_sphere.png)

I want to stress now that there is a reason why model importers like **[Assimp](https://assimp.org/)**, when told to `aiProcess_CalcTangentSpace`, generate all three components of the TBN matrix for each vertex. While yes, you can technically calculate the third component by just computing the cross product of the first two, this doesn't gurantee that the third component will point in the direction of increasing UV, which is a really useful property to have.

Finally, since the TBN matrix is *orthonormal* (all column vectors are of unit length and all of them are perpendicular to each other), its inverse is its transpose, so moving to and from tangent space is very cheap.
