---
layout: post
title: avidscript lives!
date: '2011-01-21 21:17:41'
tags:
- compilers
redirect_from:
- /avidscript-lives
- /avidscript-lives/
---

As I’ve mentioned on some recent posts, I’ve been working on developing my own compiler and virtual machine lately. And finally, it lives! It’s taken 5 months, with some language revisions along the way, but I have set out what I intended to achieve: a custom programming language, compiled down to bytecode, running in a virtual machine.

All of it was built by me from scratch in C++ using only the standard library. Admittedly, it’s not yet complete, and I’m sure there are many bugs. For now though, it’s working.

## Language design

My original aspiration had been for a concurrent state-based language, and I successfully developed a grammar and parser for that. However, working on the compiler soon made me scale it down to a simple procedural language with only scalar data-type support. I realised I had bitten off more than I could chew for my first shot at this.

I decided to try experimenting with an extremely strict language design. Variable and function declaration specifiers are required: “var” and “function”. Also, every identifier has a prefix: “t.” for type specifiers, “f.” for function names, and “v.” for variable identifiers.

This helps keep the grammar simple, although still not quite [LL1](https://en.wikipedia.org/wiki/LL_parser), and it mitigates the chances of ambiguous resolutions. I also opted to make all implicit data conversions illegal, including type promotion. For example, if you wanted to assign an int to a float, you’d have to explicitly convert the int, or else the compiler would reject it.

For example:

```
var t.int v.myNum = 15;
var t.float v.halfMyNum = t.float(v.myNum) / 2.0;
```

That may look ugly (and it is) but it’s only in implementing and using this that you realise how much leeway languages like C/C++ give you. They are normally criticised for being too rigid for beginners to learn, with strongly-typed variables and so on.

In reality, things like type promotions and implicit casts and copy constructors give more liberty than we perhaps realise. This kind of consideration becomes particularly important in low-level programming for things like games consoles, where an inadvertent conversion in a frequently-run piece of code can cost a surprising number of processor cycles.

## Example program

Here is a very simple example program which asks the user how many square numbers (n) they want to calculate. It will then output the first `n` square numbers:

```
function t.null f.main()
{
    f.consoleWrite("How many square numbers do you want to calculate? ");
    var t.int v.total = t.int(f.consoleReadLine());
    for (var t.int v.counter = 1; v.counter <= v.total; v.counter++)
    {
        f.consoleWriteLine(t.string(v.counter) + " squared = " + t.string(f.squareInt(v.counter)));
    }
}

function t.int f.squareInt(var t.int v.num)
{
    return v.num * v.num;
}
```

## Compiler structure

The compiler front-end is a fairly basic lexer/parser combination, building an [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) (AST) which implements the [visitor design pattern](https://en.wikipedia.org/wiki/Visitor_pattern).

In the back-end, code generation is done by a two-pass recursion through the tree: the first pass grabs internal function declarations and external function calls, and the second pass generates the executable bytecode. External function calls are stored in a table in bytecode. They are linked by the VM when the bytecode is first loaded.

The resulting bytecode file is structured like this:

- Header data:
  - Script type / version identifiers
  - Offset values into bytecode
- Metadata:
  - Literals table
  - Internal function table (offsets to entry points)
  - External function table (unlinked identifiers)
- Binary-encoded instructions and data

## Virtual machine structure

As with many simple virtual machines, the asvm (AvidScript Virtual Machine) is stack-based. A processing stack for parameter passing and general instruction processing, and a separate call stack for function calls.

Hypothetically, function calls are extremely fast. All that needs to happen to call a function is to push the arguments onto the processing stack (if any), push the return address onto the call stack, and jump the program counter. The VM even uses a flat variable table so there are no frames or scope issues to think about (all scope checking is done at compile-time). The return value is pushed onto the processing stack just before the function returns.

The bytecode itself is unaligned. Each instruction is 1 byte, and may or may not have some arguments of different sizes. However, the fetch-decode process is fairly fast thanks to a map of opcodes to instruction-handling function pointers.

Within each instruction handler, instruction arguments (where applicable) will be read from the bytecode. This makes the execute step have an unpredictable load/complexity, but since the whole thing is running in software there are no guarantees of execution speed anyway.

## Lessons learned

I have learned loads of stuff about what it takes to compile and run a program, including structural issues and best practices. If I designed a new compiler now, I would hugely improve the two-pass system in the back-end, making sure declarations and usages are fully resolved on the first pass. That would allow a 'middle-end' to do some interesting optimisations before the actual code-generation kicked-in.

There is an awful lot I would change about the VM as well. Currently, a lot of the data handling is somewhat disarrayed because I wasn't entirely sure where it would be getting accessed from. Now that I have a better idea of flow/data control, I would be able to crystallise the design and make things more efficient.

## What next?

Currently, there are still a lot of rough edges on the compiler and VM. For now I need to finish what I started. Not everything is tested yet, and there may be many bugs I haven't found or anticipated.

In the medium term, I would like to look at integrating the VM into a larger program, allowing data to pass between the parent program and the script. I think that will need some careful thought regarding efficiency of data transfer. I would also like to explore implementing arrays. I suspect my whole variable handling system will need an overhaul before I can do it justice, but I think it's an important stepping stone towards custom data types.

In the longer term, I want to support simple C-style structures, and possibly some very basic object-orientation (akin to the capabilities of PHP4, perhaps). And ultimately of course, I want to re-visit my plans for a concurrent state-based scripting system.
