---
layout: post
title: 'OpenGL gotcha: my particles are too small'
date: '2009-11-13 01:56:32'
tags:
- opengl
---

As graphical effects go, a particle system is one of the most versatile. It can be used to simulate things like electrical sparks, dust clouds, smoke, fire, and water. However, there is one problem which I have been annoyed by several times in the past, but just lived-with because I didn’t think it important enough to fix.

I would spend ages programming a particle effect, tweaking all the parameters to get it just right. It would look fine at first. But then, on another computer, or on a different screen resolution, or even just with a different size of window, the effect suddenly wouldn’t look right. All the particles would be too small or too big.

![particle_scaling_examples]( __GHOST_URL__ /content/images/2019/09/particle_scaling_examples.png)

You can see this problem in the image above. It shows a particle effect for a rocket thruster. The one on the left is how it should look, while the one on the right shows what happened when I increased the size of the game window.

## The Problem

First of all, what’s actually going on?

Here’s a reasonably simple fragment of code for drawing a distance-attenuated point on the origin (i.e. a point that gets smaller as it gets further from the camera):

    glPointSize(10.0f);
    float fVals[3] = {0.0f, 0.0f, 1.0f};
    glPointParameterfv(GL_POINT_DISTANCE_ATTENUATION, fVals);
    glEnable(GL_POINT_SMOOTH);
    glBegin(GL_POINTS);
    glColor3f(1.0f, 0.0f, 0.0f);
    glVertex3f(0.0f, 0.0f, 0.0f);
    glEnd();

(This obviously assumes you’re working with a perspective projection, which is the norm for 3d games.) It works fine as long as the size of your OpenGL viewport doesn’t change. For games, you usually have a single viewport covering the entire client area of your window. In full-screen mode, the viewport’s dimensions are therefore usually the same as your screen resolution. While in windowed mode, the viewport should usually change when your window is resized.

The problem is the way your projection will usually work compared to your viewport. When the viewport dimensions are increased, there are more pixels to fill. In most cases, your geometry doesn’t care about pixels — the polygons will just be expanded to fit available space prior to rasterisation. However, point size is expressed in terms of pixels (or arguably fragments, given that they can be occluded), so they are not increased to fill the space. Their route to rasterisation is more direct.

As such, when the viewport size is increased, the rendered points will appear smaller relative to other geometry. Consequently, your particle effects look wrong.

## The Solution

There are probably lots of ways to fix this problem, but I’ve gone for a very direct approach. In my current game engine, I have different types of particle renderer class, all derived from a single `IParticleRenderer` base. I’ve added a static member to that base class, called `s_fPointSizeMultiplier`, along with appropriate accessor functions.

Now, whenever the OpenGL viewport is getting updated (using glViewport), I do a quick calculation to determine the point size multiplier, as follows (variables _w_ and _h_ store the window dimensions or the screen resolution):

    glViewport(0, 0, w, h);
    IParticleRenderer::setPointSizeMultiplier( (float)h / 512.0f );

Finally, when rendering points, I change the point size function call to look like this:

    glPointSize(10.0f * s_fPointSizeMultiplier);

The result is that my particle system will now render points whose size is proportional to the size of the viewport. Changing window size or screen resolution no longer disrupts the particle effects.

## Explanation

You may be wondering two things about the multiplier calculation. First, you might be wondering why I am only using the height of the viewport. The reason is simple. Given the way projection matrices usually work, a wider viewport will not usually result in the geometry being expanded — it will simply allow more geometry to fit. The geometry will only be expanded when the viewport height increases, so the point sizes only need to be changed in that circumstance. (Try resizing an OpenGL window horizontally and then vertically, and you’ll see what I mean.)

Second, you might be wondering why I’ve chosen the fairly arbitrary value 512.0 to divide by. It just happens to be the size of window I was working with at the time for testing, so it was easier to keep all my particle effect parameters exactly as they were by using a 512×512 viewport as a reference. It doesn’t matter what scale you use for the multiplier, so long as it’s consistent.

<!--kg-card-end: markdown-->