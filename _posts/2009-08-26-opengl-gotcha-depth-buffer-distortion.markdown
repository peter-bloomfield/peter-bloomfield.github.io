---
layout: post
title: 'OpenGL gotcha: depth buffer distortion'
date: '2009-08-26 19:29:23'
tags:
- opengl
redirect_from:
- /opengl-gotcha-depth-buffer-distortion
- /opengl-gotcha-depth-buffer-distortion/
---

While I’m on the topic of OpenGL gotchas, I thought I’d mention another which caught me out a couple of years ago. I was working on some prototypes with some friends, and we encountered some strange visual glitches:

![Screenshot of visual glitches](/assets/img/migrated/depth_buffer_distortion.jpg)

Notice areas where polygons overlap: you can see a striped or checked pattern of the occluded polygon showing through. In this case, we were rendering 2d graphics with a perspective (3d) view so that we could make some interesting visual effects. However, the problem can manifest just as badly in full 3d.

## The problem

You would be forgiven for thinking any of the following were at fault:

- Polygons are too close together
- Depth equation is incorrect
- Depth buffer is too small (not enough bits)
- Near and far clipping planes are too far apart

In reality, the problem was actually that the near clipping was too close to the camera. If you want to see this for yourself, just try modifying existing code to put the near clipping plane extremely close to 0.

## Why does this happen?

When your scene is going to be rendered, all the geometry gets passed through the various coordinate spaces until it is mapped nicely into your viewing frustum. The frustum is effectively transformed into an orthogonal cuboid. One side of it becomes your rendering viewport.

At this point, viewport dimensions are mapped onto the X and Y axes. The scale of these axes is usually determined by something like window size or screen resolution. Meanwhile, the depth buffer is mapped onto the Z axis. The size of the depth buffer is usually determined when initialising OpenGL. 24 bits is quite a common value. It could hypothetically represent around 16.7 million different depths in between your near and far clipping planes.

You might expect the depth buffer to be uniformly mapped along the Z axis of this viewing box. However, the mapping is usually denser towards the near clipping plane and sparser towards the far clipping plane. This is to allow for more accurate rendering for objects which are closer to the camera and which are therefore more noticeable.

As a result, if your near clipping is at or near 0 depth then your depth buffer mapping is almost entirely clustered at that point. In other words, you end up with very few bits left for the rest of your viewing frustum. The consequence is such low precision depth representation that it looks like your depth buffer is being distorted or corrupted.

## The solution

As a general rule, don’t put your near clipping plane at depth 0. Depth 1 is usually a good starting point, but it depends somewhat on your application.
