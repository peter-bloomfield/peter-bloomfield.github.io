---
layout: post
title: 'PHP quirk: shell commands breaking on Windows'
date: '2009-09-11 13:04:01'
tags:
- php
- windows
redirect_from:
- /php-quirk-shell-commands-breaking-on-windows
---

I’ve been wrestling with many compatibility quirks in the open source [SLOODLE project](http://www.sloodle.org) lately, and this one certainly deserves some kind of prize for awkwardness. It seems that some perfectly valid shell commands will simply not work when executed from within PHP on Windows.

## Background

Some of our code in SLOODLE needs to execute a shell command to make use of “[ImageMagick](http://www.imagemagick.org)“. (This is actually only as a fall-back in case the MagickWand extension isn’t present). The basic structure of the shell command is simple:

    convert src dst

It works fine for simple cases. However, if there are any spaces in the path to the “convert” program, or in the paths specified by “src” and "dest”, then the paths need to be enclosed in quotation marks. That’s almost always the case on Windows because it puts most software in a directory called “C:\Program Files”.

For some reason, if the program being executed _and_ one or more of the arguments are enclosed in quotation marks, then it won’t work from PHP on Windows. The same command works fine on the Windows command line though.

From within PHP, the “exec” command gives a return code of 1 and no output, and the “shell\_exec” command doesn’t seem to do anything at all. No warnings, notices, or error messages are reported.

## The solution

After some digging around, I found a few people using the Windows “start” command to solve some problems. It tried it, and it works! All you need to do is prepend some stuff to your shell command if you’re running on Windows. Our command from above ends up looking something like this:

    start /B "" convert src dst

The empty quotation marks after `/B` are irrelevant for our purposes, but they are required nonetheless. You can find out more about the `start` command by typing `start /?` on your Windows command line.

Obviously it’s worth noting that it won’t work on Linux, so you need to check the operating system in PHP before modifying your shell commands. You could actually put it in a handy function like this:

    function makeWindowsCompatible( $cmd )
    {
        if (substr(php_uname(), 0, 7) == "Windows")
            return 'start /B "" '.$cmd;
        return $cmd;
    }

Pass your existing shell commands into that function, and the result will be appropriately modified for Windows if necessary. You would use it something like this:

    $cmd = 'convert src dest';
    echo shell_exec( makeWindowsCompatible($cmd) );

<!--kg-card-end: markdown-->