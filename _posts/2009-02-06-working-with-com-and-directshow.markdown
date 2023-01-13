---
layout: post
title: Working with COM and DirectShow
date: '2009-02-06 22:48:30'
tags:
- cpp
- windows
---

I’m doing some work with webcams just now, and in the pursuit of efficiency, I am turning to DirectShow. That’s the part of Microsoft’s [DirectX](http://en.wikipedia.org/wiki/DirectX) which handles things like videos and webcams.

I've hardly used DirectX before, except briefly in the first year of my undergraduate degree, so it’s fairly new ground for me. However, I quickly found that it really is well worth spending time getting familiar with the technology as it can be amazingly useful.

## Getting used to COM

The Component Object Model (COM) has a bit of a bad reputation for being complex. Having not really used COM or DirectX significantly before, I was a little intimidated at first. However, after a couple of hours reading through the DX9 SDK documentation, it’s really not that bad.

The fact that you have pointers-to-pointers can be a little off-putting. It’s also annoying that COM seems to circumvent or wrap lots of basic C++ conventions like memory allocation. Once you understand some of the principles though, you are in good stead to tackle a wide range of DirectX tasks. COM is apparently also used in other areas of Windows programming, although I have not encountered it elsewhere myself yet.

## Objects and interfaces

The basic principle of COM is that you have various kinds of objects, and each object has various interfaces to it. Each interface provides a well-defined set of functions for manipulating the object, and the same interface can be implemented by any number of different objects.

For example, let’s imagine you have a `Person` object and a `Dog` object, and you want to control the way they walk. The COM wouldn’t let you control that directly, but it might provide a movement interface, such as `IMove`.

You would simply request an `IMove` interface for your `Person`, and use it to make the person walk, e.g. using `IMove::WalkForwards()`. You could do the same for the `Dog`.

As far as your code is concerned, it is the same interface, but it has been implemented differently depending on whether you are using it on a `Person` or a `Dog`. For some programmers, the confusing thing is that the object doesn’t implement the interface in the way you might expect if you use a language like Java. Rather, interfaces are effectively implemented _for_ specific objects.

If you are familiar with the [Model-View-Controller](http://en.wikipedia.org/wiki/Model-view-controller) design pattern, then think of the objects as the models and the interfaces as the controllers.

It is also worth noting that there are two main ways of actually creating objects with COM. Firstly, you can directly create an object, specifying what kind of interface you want for it initially. Or secondly, you can use an indirect method, which usually means calling a helper function which does extra configuration. It then gives you a pre-determined interface type. Either way, you can request further interface types after the object is created.

## Filter graphs in DirectShow

After getting up to speed on COM, I looked through some of the DirectShow documentation. There is an awful lot to take in, but the basic principle breaks down into “filters”.

A filter is something which somehow operates on audio and/or video data. You would connect a series of these filters together into a tree-like graph structure to achieve your goal. Many of the complexities are handled by the filter graph manager which DirectX provides. For common tasks, it can even setup the graph entirely on its own.

Some simple examples of filters are given in the documentation. For example, you might start with a source filter to read data from an AVI file (Audio/Video Interleave). You then have a splitter filter to separate the audio and video parts into different parts of the graph.

Next, you need an audio decoding/decompression filter or similar to output the audio, and something similar for the video feed. Finally, you need a video render filter. Everything is tied and synchronised together by the filter graph manager.

## Playing a Video

I went through an example in the documentation of using DirectShow to play a video, and it found it remarkably straightforward after having spent some time familiarising myself with everything above. I think a common mistake with this is jumping straight to the source code. It’s easy get bogged down by not understanding _why_ certain things work in certain ways.

The results came together remarkably easily. I created a console application which sets up DirectShow, and then plays a video from a file in a small window. You can see a screenshot of it below. [Click here view the source code](https://gist.github.com/peter-bloomfield/70501ddff33abcc379910a0cdc3db26a). It was developed in MS Visual Studio 2003.

![video_player]( __GHOST_URL__ /content/images/2019/09/video_player.jpg)

Two things you need to note: in order to compile it, you will need the link in with a couple of DirectShow libraries, “Strmiids.lib” and “Quartz.lib”. Also, remember to put in the path/filename of a video file (preferably AVI or WMV) on line 66.

<!--kg-card-end: markdown-->