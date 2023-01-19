---
layout: post
title: Using GDI+ in C++ Win32 programming
date: '2009-05-02 02:08:57'
tags:
- cpp
- windows
redirect_from:
- /using-gdi-plus-in-cpp-win32-programming
- /using-gdi-plus-in-cpp-win32-programming/
---

If you do any Win32 programming in C++ then I strongly recommend you learn about GDI+. Although it’s been around for a while now, it doesn’t seem to be well known. It can be great to have on hand even just to illustrate tests and prototypes though.

As it’s object-oriented, it’s much nicer and easier to use than the basic C-style GDI that used to be the norm. It also provides a lot of additional functionality which otherwise was not possible (or at least not easy) with the regular GDI functions alone. For example, proper alpha blending, matrix transformations, file input/output, and loads more. It’s quite easy to setup too.

## Unicode

One thing to be aware of first: GDI+ requires Unicode to be enabled. That means that all string literals need to be preceded by `L` or encased in the `TEXT(..)` macro. It also means that you might find you need to change any string classes or functions. It can be a nightmare to port existing code, but it’s alright once you get there (or if you’re starting from scratch).

## Configure the Project

I’m using Visual C++ Express Edition 2008, which is free to download and use. The best thing to do is setup a simple windows application first. Just create a basic window, and your regular message pump/handler.

Next, you need to make sure Unicode is enabled for your code. To do that, go into your project properties page, select “C/C++” → “Preprocessor”, and beside “Preprocessor Definitions”, add “UNICODE”. Do this for Debug and Release modes, or whatever your configurations are.

After that, you need to link to the Gdiplus library. Still in Project Properties, go to “Linker” → “Input”, and beside “Additional Dependencies”, add “gdiplus.lib”. (Once again, do it for all configurations.)

## Initialisation and cleanup

Now you need to add the code to initialise and cleanup the GDI+ system. Put the following code somewhere near the start of your WinMain function (before you create any windows):

```cpp
GdiplusStartupInput gdiplusStartupInput;
ULONG_PTR gdiplusToken;
GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, NULL);
```

Next, put the following code somewhere near the end of your WinMain function, after all other GDI+ objects have been deleted or fallen out of scope:

```cpp
GdiplusShutdown(gdiplusToken);
```

Finally, somewhere near the top of your source code file(s) where you will be using GDI+, you’ll want to put this:

```cpp
#include <gdiplus.h>
using namespace Gdiplus;
```

## How to use it

You’ll usually want to use GDI+ in your window’s “paint” event (although it can be used to write out to files too if you want). The main class you’ll be working with is the `Graphics` class which handles most of your drawing.

You have to start by getting a `Graphics` object linked to the device context of your window so it can draw to it safely. There’s lots of ways to handle a paint event, but I’ll follow my preferred approach here (remember to make sure you’re window area is invalidated before doing this):

```cpp
// Assuming you've got your window handle in "hWnd":
PAINTSTRUCT ps;
HDC hdc = BeginPaint(hWnd, &ps);
Graphics g(hdc);
```

Just like with the regular GDI, you draw and paint using pens and brushes, but thankfully these are much easier. We will fill in a rectangle with a red brush, and draw a green circle inside it:

```cpp
SolidBrush redBrush(Color::Red);
Pen greenPen(Color::Green, 2.0);
g.FillRectangle(&redBrush, 20, 20, 100, 100);
g.DrawEllipse(&greenPen, 30, 30, 80, 80);
```

Finally, we need to tell our window to finish painting now (this bit isn’t GDI+):

```cpp
EndPaint(hWnd, &ps);
```

## Documentation

As you can see, it’s fairly easy to setup and use GDI+. I recommend looking at [Microsoft's GDI+ documentation](https://learn.microsoft.com/en-gb/windows/win32/gdiplus) to help you get going.
