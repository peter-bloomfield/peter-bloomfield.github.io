---
layout: post
title: Path error when building with Conan and CMake
date: '2019-07-18 19:50:00'
tags:
- cmake
- conan
- windows
---

I recently encountered a mysterious path error when attempting to build a C++ project on Windows using [Conan](https://conan.io/) and [CMake](https://cmake.org/). The main error message wasn’t particularly helpful though:

     ERROR: conanfile.py (my-project/1.0@None/None): Error in build() method, line 71
         cmake.configure(source_folder=".")
         ConanException: Error 1 while executing cd C:\path\to\repo\build && cmake -G "Visual Studio 15 2017 Win64" ...
    20190718 15:00:46 : ERROR : Problem: conan_build: 1
    20190718 15:00:46 : ERROR : Exit: 1

I’ve edited the message for brevity and to avoid sharing details of exactly which project it was. The main point is that Conan encountered the error when it was trying to run a shell command. The shell command was attempting to enter a build directory and invoke CMake. I tried to run the command manually and it worked correctly so something strange was going on behind the scenes.

## The actual error

Further back in the log, there was another little error message sitting quietly among a lot of other build information:

> The system cannot find the path specified.

This is exactly the error message you get if you attempt to enter a non-existent directory in the Windows command prompt. As such, it looked like the “cd” part of Conan’s command was failing. However, the build directory definitely existed. I tried doing a clean checkout in an entirely different directory with full read/write access for all users, but I still had the same problem.

The build had worked on many previous occasions using exactly the same repository and commands, and it still worked correctly for everyone else on the team. As such, something on my system must have changed. I had been working on a few other projects for several weeks so I couldn’t remember everything I had done since the last successful build.

## The investigation

I initially tried reinstalling and updating lots of things in an effort to fix it, including Conan, CMake, Git Bash, Visual Studio, and Python. None of these made any difference at all.

Next, I tried installing and running [Conan from source](https://docs.conan.io/en/latest/installation.html#install-from-source), putting a [manual breakpoint](https://poweruser.blog/setting-a-breakpoint-in-python-438e23fe6b28) in our conanfile.py, and stepping through the code using [PDB](https://docs.python.org/3.7/library/pdb.html). It took me through various functions in Conan, including [CMake](https://github.com/conan-io/conan/blob/develop/conans/client/build/cmake.py).configure(), CMake.\_run(), [ConanFile](https://github.com/conan-io/conan/blob/develop/conans/model/conan_file.py).run(), and finally [ConanRunner](https://github.com/conan-io/conan/blob/develop/conans/client/runner.py).\_\_call\_\_(). That last one is where the shell command was executed so I was able to confirm exactly what was being passed to the OS.

I couldn’t see any problems in the code, but I started to suspect that the shell command was somehow being misinterpreted. I thought perhaps the entire command, including the double ampersand and everything after it, were being interpreted as the path of the build directory. That would certainly have caused the “cd” part to fail.

To investigate this, I tried modifying the Conan source code to run the command differently. I very quickly realised that any command I ran resulted in the same path error: “The system cannot find the path specified”.

This suggested that Conan’s command wasn’t being executed at all. It seemed like the system was trying (and failing) to execute something else beforehand. This made me think of files like .bashrc and .bash\_profile which can execute custom commands when the bash shell starts on Unix-type systems.

I’m aware of various equivalents for other shells like zsh, but I wasn’t familiar with any equivalent for the Windows shell. With a little Googling, I quickly ran across a likely candidate on the Super User Stack Exchange:

- [https://superuser.com/questions/727316/error-in-command-line-the-system-cannot-find-the-path-specified](https://superuser.com/questions/727316/error-in-command-line-the-system-cannot-find-the-path-specified)
- [https://superuser.com/questions/144347/is-there-windows-equivalent-to-the-bashrc-file-in-linux](https://superuser.com/questions/144347/is-there-windows-equivalent-to-the-bashrc-file-in-linux)

In short, the Windows registry can contain commands which run automatically whenever a Windows shell is started. This can happen whether you launch a command prompt manually or run a shell command from within another program. The commands can be placed under these keys:

- HKCU\Software\Microsoft\Command Processor\AutoRun
- HKLM\Software\Microsoft\Command Processor\AutoRun

## The actual problem

I found an AutoRun entry under HKCU which was attempting to run some setup code for [Conda](https://conda.io/). I had tinkered with Conda briefly a few weeks earlier when I was working with some Python code. However, I eventually decided not to use it so I uninstalled it and went back to a standard Python installation. Apparently, the uninstaller did not remove the key. With Conda no longer on the system, the command was failing.

## The solution

The Conda command was the only thing in the AutoRun key. I emptied it and my Conan builds started to work again.

## Conclusion

This was an obscure and frustrating problem which took me a couple of days to solve in total. I’m glad I got there in the end, and it’s certainly useful to know about the AutoRun key in the registry for future reference. I’m disappointed that the Conda uninstaller left it behind though. It’s possible that I did something silly to cause it, but unfortunately I can’t remember as I was experimenting with several things while working on another project.

I’m also surprised that the path error didn’t show up whenever I launched the command prompt normally. That probably would have tipped me off about the problem much sooner.

I think the main lesson to learn from this is to be careful with your build environment. It can sometimes be quite a delicate ecosystem which is easy to disrupt, especially when working with a language like C++. If you need to experiment with something, consider doing it on another computer or in a virtual machine.

