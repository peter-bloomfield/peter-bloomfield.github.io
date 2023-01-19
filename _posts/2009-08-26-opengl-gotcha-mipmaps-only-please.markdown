---
layout: post
title: 'OpenGL gotcha: mipmaps only please!'
date: '2009-08-26 18:22:07'
tags:
- opengl
redirect_from:
- /opengl-gotcha-mipmaps-only-please
- /opengl-gotcha-mipmaps-only-please/
---

Here’s an OpenGL gotcha which caught me out for quite a while. I was writing code to load textures, but it only worked if I was using the GLU function to build mipmaps. Loading standard textures kept failing. After lots of frustration, I eventually found my mistake.

## Background

My code used the [FreeImage](http://freeimage.sourceforge.net/) library to load the image files from disk. I was then rescaling to powers-of-two dimensions, and checking that the input format was compatible. Finally, I was loading the texture using `glTexImage2D()`.

I didn’t care about mipmaps at this early stage so I loaded the texture at level 0. I’d made sure the minification filter was set to `GL_LINEAR` (which doesn’t use mipmaps).

Everything seemed fine in the code, and yet the textures were not being rendered. All I could see were coloured polygons. The only way I could get the textures to render was if I switched my `glTexImage2D()` function call for `gluBuild2DMipmaps()`. I double-checked that my texture filters were correctly setup, and sure enough I had the following code in my OpenGL initialisation function:

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
```

## The problem

It took a long time, but after some digging, I eventually realised my mistake. I had assumed that the texture parameters set by `glTexParameter*()` were applied to the texture target (in this case, `GL_TEXTURE_2D`). However, they are actually per-texture properties. That is, the properties are applied only to the texture object which is _currently bound_ to that target at the time the texture parameter function is called.

## The solution

It’s easy really. Instead of specifying the texture parameters in my OpenGL initialisation function, I specified them in the texture loading function, immediately after my call to `glTexImage2D()`. Everything worked fine after that.
