---
layout: post
title: Improve C++ performance in debug mode with inline expansion
date: '2012-07-28 20:37:28'
tags:
- cpp
- debugging
- visual-studio
---

When you’re dealing with an intensive real-time application, such as a game or simulation, a common problem is that debug builds in C++ can run much slower than release builds. This difference in run-time behaviour means it can be hard to reproduce and analyse bugs and other problems. There are several things you can do to improve it, and one which helped me recently was enabling inline expansion.

A quick warning though: it won’t improve performance in all situations, and it can actually hinder debugging. For performance-critical code, you should first try manually optimising your algorithms.

# When is it useful?

This technique might be useful if your program is calling lots of small functions extremely frequently — e.g. several thousand times per frame. Examples are accessor methods and overloaded operators. Ideally, you should minimise your use of these in very intensive code. However, the realities of writing readable code on a budget / deadline sometimes make that difficult.

# What is inline expansion?

Each time your program calls a function, it incurs a small overhead as it handles the registers and stack frame etc. The vast majority of the time, this overhead is negligible. However, when repeated thousands of times in quick succession, it can start to add up.

If the compiler inlines a given function then it avoids the call overhead altogether. It effectively copies the function’s body to wherever it’s called from. This means the resulting program will be bigger because the function body is duplicated in several places. However, the program can fly through those blocks of code without having to jump or manipulate the call stack.

# Why isn’t it enabled already?

Release build configurations usually enable inlining, among many other optimisations. Debug builds typically disable it for a very good reason — you can’t always step-through the code or watch the variables of a function that’s been inlined. All the other code should be fine though, so you have to make a judgment call yourself on the tradeoffs in your own program.

I would recommend creating a separate build configuration, called something like “Debug Fast”. This allows you to switch back to regular debugging easily, but get the extra speed when you need it.

# What functions would be affected?

Any good compiler is capable of deciding which functions to inline and which to leave alone. You can typically opt to have the compiler make this decision on its own, or force it to inline only the functions you specifically declare using the “inline” keyword. I recommend the latter option where possible to ensure you minimise the impact on your debugging workflow.

As mentioned above, accessors are typically good candidates for inlining, as the code is often only a single return or assign statement. Note that this also applies to the STL as well. The code I’ve been using recently makes extensive use of [`std::vector`](https://en.cppreference.com/w/cpp/container/vector), and accesses it using the subscript operator `[]`. This was identified as a major performance bottleneck so enabling inline expansion made a huge difference to the program’s framerate.

# What performance benefits can I expect?

The results depend entirely on your own code. You might find it makes absolutely no difference whatsoever, or you might find it increases your framerate tenfold. Whenever you have performance issues though, you should consider profiling your code to see what’s actually slowing you down before making any big changes. This technique will only work if significant bottlenecks exist in one or more functions which can be inlined.

# Setting it up in VS2010

## Create a new build configuration

This step is optional, but highly recommended. Creating a separate build configuration allows you to switch back to your original debug configuration when necessary.

1. Click the “Build” menu
2. Click “Configuration Manager”
3. Click the “Active solution configuration” drop-down
4. Select “\<New\>”
5. Type in a name, such as “Debug Fast”
6. Under “Copy settings from” select “Debug”
7. Tick the checkbox which says “Create new project configurations”
8. Click OK
9. Click Close

You will now be able to switch to the fast debug configuration easily when you need it.

## Changing the project settings

You’ll need to follow these steps for each project you need to improve debug performance on:

1. Right click the project (not the solution itself) in the “Solution Explorer” pane
2. Click “Properties”
3. Under “Configuration”, select the debug configuration you want to use (i.e. the “Debug Fast” configuration you just created, if applicable)
4. Expand the “C/C++” group
5. Click the “General” category
6. Beside “Debug Information Format”, select “Program Database” (not “Edit and Continue”)
7. Click the “Optimization” category
8. Beside “Inline Function Expansion”, select “Only \_\_inline (/Ob1)”
9. Click OK

Step 6 is important because function inlining is not compatible with “Edit and Continue”.

At step 8, you can choose “Any suitable (/Ob2)” if you need to improve performance further. However, I recommend sticking with “Only \_\_inline” if possible as it will avoid inlining things which might not be needed. It’s not always possible to debug functions which have been inlined.

## Declare functions inline

If you identify specific small functions which may be slowing you down then declare them as inline. For cases where a function is only used in the source file where it’s defined, just precede its prototype with the [`inline` specifier](https://en.cppreference.com/w/cpp/language/inline). If the function is called from any other file then you’ll typically need to move the definition into the header file where it’s declared.

If you encounter linker errors when you’re doing this (such as “unresolved external symbol”) then it may mean you didn’t move a function into the header file. It may also mean you moved it into the wrong header file, or got the declaration slightly wrong.

## Build and run

Now that you’ve setup your build configuration, you simply need to rebuild the program and run it. A full rebuild is usually required, so it might take a while if you have a large program.

## External libraries

You might find that your performance bottlenecks actually exist in your dependencies rather than in your own code. If that’s the case, this technique will only work if you can reconfigure and rebuild the problematic library using the same technique.
