---
layout: post
title: Fixing LNK4098 in Visual C++
date: '2010-12-14 02:31:28'
tags:
- visual-studio
- cpp
---

Like many C++ linker errors and warnings, LNK4098 is a little vague and cryptic at first. It comes in different flavours, but it will often have other errors with it, such as LNK2005 (“symbol already defined”). Here’s one I got recently in VC++ 2010 Express:

```
LINK : warning LNK4098: defaultlib 'LIBCMTD' conflicts with use of other libs; use /NODEFAULTLIB:library
```

You might find “LIBCMTD” replaced with other things, such as “LIBC” or “LIBCD”. I recommend taking a little time to understand what’s going on, as knowledge really is power for programmers, and it will hopefully help you avoid similar problems in the future.

## When would I get the LNK4098 warning?

You should only find this warning appearing when your project has dependencies. For example, you might be using an external library such as a .lib or .dll file. Alternatively, you might be working on two or more C++ projects in one solution, and linking them together internally.

This should normally work without any problems. However, LNK4098 is telling you that your configurations aren’t consistent — specifically, you are trying to use different default run-time libraries in different parts of your solution.

## What are default run-time libraries?

Unlike many simpler programming languages, much of the C++ ‘core’ functionality can come in different shapes and sizes. This is because C++ code is compiled to run natively so there is no Virtual Machine or similar program to make execution decisions at run-time. When you’re dealing with Visual C++, your default run-time libraries can be broadly divided into single-threaded and multi-threaded variants. Multi-threaded is a little bigger and slower, but provides thread-safety in case it’s needed.

Each one also has a debug and non-debug (or ‘release’) version, depending on whether you are compiling for testing or release. And finally, the multi-threaded libraries can also come in static and dynamic varieties. Static requires that you compile all the used library code into your project, making your final executable bigger but (hypothetically) more portable. The dynamic version results in a smaller executable but relies on having the appropriate DLL files present on the target system.

In other words, there are 6 options:

- `LIBC`: Single threaded
- `LIBCD`: Single threaded debug
- `LIBMCT`: Multi threaded
- `LIBCMTD`: Multi threaded debug
- `MSVCRT`: Multi threaded DLL
- `MSVCRTD`: Multi threaded debug DLL

## How do I fix it?

The short answer is that your project and all its dependencies **must be compiled against the same default run-time libraries**.

If your dependencies are external (e.g. you downloaded a library from the web) then try to find out what run-time libraries were used when they were compiled. Put that into your project’s configuration, and try to recompile. (You could also just guess what configuration to use — there aren’t many possibilities). If all else fails, and if the external library you are using is open source, then try compiling it yourself so that you know what setting was used.

If your dependencies are internal (i.e. different projects in your solution) then just set the default run-time library to be the same in all projects. I recommend using the multi-threaded DLL versions. Make sure to specify debug or non-debug appropriately, depending on which build configuration you are editing.

## How do I find the configuration option?

Open up the Project Properties dialog for the project you want to configure. Try right-clicking it in the “Solution Explorer”, then clicking “Properties”. You should find the “Runtime library” option in “Configuration Properties &rarr; C/C++ &rarr; Code Generation” section.

Remember that you will need to set the option for each build configuration (typically Debug and Release), and possibly for each target platform too. You can select the build configuration from the top-left of the Project Properties dialog. Make sure you only use the debug version while in debug mode as it tends to generate much bigger and slower code.

## What about the linker’s advice?

You will notice that the linker often suggests something like “use /NODEFAULTLIB:library” at the end of the warning. It’s best to ignore the suggestion as it’s usually a bad idea. It is advising you to leave out some parts of the default run-time libraries in the hopes that you won’t need them. It _might_ work in the short-term (if you’re lucky), but could easily come back to haunt you later. It’s much better to solve the source of the conflict than attempt a patchy workaround.

Admittedly, there may occasionally be situations where this kind of workaround is absolutely necessary. It should never be the first resort though.

## Where can I get more information?

This is a really common problem. A quick search online for “LNK4098” should give you loads of information.
