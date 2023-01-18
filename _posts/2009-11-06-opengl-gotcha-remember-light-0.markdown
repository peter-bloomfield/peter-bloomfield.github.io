---
layout: post
title: 'OpenGL gotcha: remember light 0'
date: '2009-11-06 00:14:20'
tags:
- opengl
redirect_from:
- /opengl-gotcha-remember-gl_light0
---

I was experimenting with some simple scene geometry and lighting in OpenGL today, and I stumbled into the same pitfall I’ve stumbled into several times before. (You’d think I would have learned by now!) While moving around the scene, I found that the lighting on surfaces appeared to change depending on my camera angle. If I looked at a polygon square-on, it seemed much brighter than if I was looking slightly away from it.

There are two common reasons for this problem.

## Reason 1: Lights, Camera, Action…?

The rookie mistake is to specify your lights before you’ve specified your camera angle. When you specify a light position in OpenGL, it’s modified by the model-view matrix in the same way as geometry. As such, the order for operations should typically be:

- Reset your matrix (using `glLoadIdentity()`).
- Specify your camera position/angle (e.g. using `gluLookAt()`).
- Set your light positions and other parameters (using `glLightf*()`).
- Render your geometry.

However, in this case, that wasn’t the mistake that I had made.

## Reason 2: Ambience

The mistake I made was careless use of light 0 (`GL_LIGHT0`) as an ambient-only light source. Here’s the code I was using just before rendering my geometry:

    glEnable(GL_LIGHT0);
    glLight4f(GL_LIGHT0, GL_AMBIENT, 0.2f, 0.2f, 0.2f, 1.0f);

(Note: `glLight4f()` is not a standard OpenGL function. It’s a wrapper I wrote to make lights a little easier to specify. It simply puts the 4 float values into an array and then calls `glLightfv()`.)

The problem is that light 0 is unique in having a default diffuse setting of fully bright white (1,1,1,1). In contrast, all other lights have a default diffuse setting of totally dark (0,0,0,1).

All lights also have a default position of 0,0,1,0, which would effectively place it as a directional light, facing forward, from just behind the camera. Given that the default setting was obviously in the system _before_ my camera was specified, it had the effect of moving around with my camera, and pointing the same direction.

The “w” parameter in the light position, which is the 4th component, determines whether the light is directional or whether it’s a point source; values 0 and 1 respectively.

## The Solution

The most obvious and effective solution is quite simple:

    glLight4f(GL_LIGHT0, GL_DIFFUSE, 0.0f, 0.0f, 0.0f, 1.0f);

Just add that line after the light is enabled, or perhaps even do it once in your initialisation function if you’d prefer. That prevents light 0 from having any diffuse influence as a point/directional source.

If you are specifying specular attributes in your polygon materials, then you’ll need to do the same for light 0’s specular colour. This is because its default specular colour is 1,1,1,1 as well.

<!--kg-card-end: markdown-->