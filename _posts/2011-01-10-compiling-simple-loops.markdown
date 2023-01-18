---
layout: post
title: Compiling simple loops
date: '2011-01-10 22:53:27'
tags:
- compilers
---

I program weird stuff to unwind. Lately, my pet project has been a custom programming language which I’m calling “avidscript”. It's a somewhat C-style procedural language which compiles down to bytecode.

My intention is that it will execute in a stack-based virtual machine (VM) which is integrated into a game framework. In the long term, I have drawn up some specifications to develop it into a concurrent state/event-based language which will hopefully be useful for programming game AIs.

At the moment, I’m working on the low-level code generation. This is part of the compiler "[back-end](https://en.wikipedia.org/wiki/Compiler#Back_end)". The front-end parses the source code, turning it into an Intermediate Representation (IR) of the program. The back-end then turns the IR into final compiled code. In this case, bytecode for a custom VM.

However, it could equally be native machine code. I found the bytecode representation of simple loops quite interesting, particularly from an optimisation standpoint, so I thought I’d share my findings here.

## ‘while’ loops

This is a very common loop in C-style languages. From a programmer’s point-of-view, it is perhaps the simplest. In source code, it typically looks something like this:

```cpp
while (c)
{
    // statements to repeat...
}
```

It's fairly easy to understand: while some condition `c` is true, keep repeating the list of statements. Compiling this down to bytecode results in pseudo-assembly language something like this:

```
PUSHVAR [address of variable c]
JUMPIFZERO [loop end]
[instructions to repeat]
JUMP[loop start]
```

This looks fairly simple. First, you push the value of the condition variable onto the stack (or otherwise evaluate whatever other condition you might have). Then if that condition is false, you jump execution to the first bytecode location immediately following the loop body. Otherwise, carry on and execute the loop body. Finally, jump back to the start of the whole structure again, where you’ll push c back onto the stack to re-evaluate it.

Generating the code for this is actually a little harder than it looks. Until you generate the code for the loop body, you don’t necessarily know how much space it will take up. As a result, when you generate the “JUMPIFZERO” command, you actually have to leave the destination address empty. After you’ve finished generating all the code for the whole structure, you go back and populate it with whatever bytecode address will be executed next.

## ‘do-while’ loops

I find that this kind of loop tends to be used less often now. I’m not sure why, although perhaps because it intuitively seems less applicable and/or more complicated than a ‘while’ loop:

```cpp
do
{
    // statements to repeat...
}
while (c);
```

At first, I expected it would be at least as complex in bytecode as the ‘while’ loop. In actual fact, it is simpler:

```
[instructions to repeat]
PUSHVAR [address of variable c]
JUMPIFNONZERO [loop start]
```

It’s not much of a difference, but we’ve condensed the loop mechanics by one instruction, and the more instructions you can save the better when compiling code. The other advantage is that you don’t have to go back and replace a placeholder jump instruction like we did on the ‘while’ loop. That makes the code output very slightly more efficient because it saves 3 extra file operations (i.e. move, write, move).

## Revisiting ‘while’ loops

Having written the code-generator for both types of loop, I realised we could actually re-structure the ‘while’ loop a little to reduce the number of instructions on each iteration:

```
JUMP [condition]
[instructions to repeat]
PUSHVAR [address of variable c]
JUMPIFNONZERO [loop start]
```

Essentially, this turns the ‘while’ loop into a ‘do-while’ loop, but the first jump skips the loop body on the first iteration. The jump at the end only jumps back to the start of the loop body. The result has the same mechanics as our existing ‘do-while’ loop, but only requires a single instruction to evaluate the condition and to jump if necessary.

## Does it matter?

From a practical standpoint, you might wonder why there’s a difference worth caring about. The difference is that any processor or virtual machine requires some kind of [fetch-execute cycle](http://en.wikipedia.org/wiki/Instruction_cycle). That is, it fetches an instruction and then executes it.

With processor speeds as fast as they are, you will usually find that the fetch step is considerably slower than the execute step. As such, the fewer fetches we can do the better, within reason. The instruction set still needs to have a reasonably concise specification otherwise the decode lookup in the fetch step could become too cumbersome.

Beyond that, I would also argue that it’s essential that tools such as compilers stay as efficient as possible. Many programmers are not efficiency-conscious, or they just don’t have the time or manpower to optimise their code. This is why so much work goes into the ‘middle-end’ of compilers, which find ways of optimising source code automatically.

If you have ever worked on a computationally-intensive program, you may have noticed that a debug build usually runs much slower than a release build. This is partly because optimisations are usually disabled in debug configurations.
