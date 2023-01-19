---
layout: post
title: 'OpenGL gotcha: scale your normals!'
date: '2009-10-31 01:22:44'
tags:
- opengl
redirect_from:
- /opengl-gotcha-scale-your-normals
- /opengl-gotcha-scale-your-normals/
---

My OpenGL blunder this evening involved forgetting to make sure my normals were scaled correctly.

I had a scene with a single point light source and several small rotating cubes. Everything looked fine. I had also written a function to draw a 1×1 plane in the scene, and that looked fine too. Next, I applied a scaling transformation in my plane-drawing function (using `glScalef()`) to make it 10×10, but for some reason the plane was then totally dark.

I tried moving and rotating the plane to see if it would catch the light at some other angle, but it wouldn’t. Eventually, I spotted my mistake.

## The problem

What I had neglected was to specify a normal when drawing the plane. The last known normal was presumably a totally valid unit normal at the time. However, it was obviously specified _before_ I had applied the 10x scaling transformation. As a result, the effective normal was one tenth the size it should have been for the plane, meaning the lighting equation only yielded 10% of the expected light.

When you specify a normal, it gets transformed by the current model-view matrix in the same way as vertices. That means any normal specified _before_ a scaling transformation is probably not valid, unless you accounted for the scaling when you specified the normal.

## The solution

The ideal solution to this is always to specify the normal immediately before the vertex/vertices it applies to.

The easier solution to this problem is to tell OpenGL to normalize all normals automatically. You can do this selectively by enabling/disabling it at certain points, or you can enable once at the start of your program and forget about it. It’s unfortunately not very efficient, so the latter option isn’t preferable.

```cpp
glEnable(GL_NORMALIZE);
// drawing code...
glDisable(GL_NORMALIZE);
```

You can also opt for `GL_RESCALE_NORMALS` instead which is more efficient. However, it will go horribly wrong if your model-view matrix has a non-uniform scale transformation. (Basically, only use it if you’ve scaled your matrix by the same amount on all 3 axes.)
