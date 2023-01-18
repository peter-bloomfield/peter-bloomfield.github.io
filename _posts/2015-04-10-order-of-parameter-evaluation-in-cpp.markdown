---
layout: post
title: Order of parameter evaluation in C++
date: '2015-04-10 20:46:24'
tags:
- cpp
redirect_from:
- /order-of-parameter-evaluation-in-cpp
---

The low-level details of how data gets passed into a function are often overlooked by programmers. We obviously care about passing parameters by value vs. reference and by copy vs. move, but it’s easy to ignore anything deeper than that.

With C++ in particular, this can cause an unexpected problem regarding the order in which things actually happen. In this post, we’ll look at what can go wrong, and how to deal with it.

## Simple example

Let’s say we’ve got a function which needs to be given two numbers. However, we don’t have those numbers stored in variables yet. Rather, we need to call two other functions to get those values. For example:

```cpp
#include <iostream>

void doStuff(int a, int b) { }

int getA()
{
    std::cout << "A";
    return 1;
}

int getB()
{
    std::cout << "B";
    return 2;
}

void main()
{
    doStuff( getA(), getB() );
}
```

What is the execution path in `main()`?

Obviously `getA()` and `getB()` must both be called before `doStuff()` can be called. That much absolutely must be true. However, it’s also tempting to make the assumption that `getA()` will be called before `getB()`.

In reality, this may not be the case. If you run the program, the output is likely to be “BA”, indicating that the parameters were evaluated in reverse order. Note that it depends on your compiler though. In theory, you may see different results depending on which one you use.

## Why does this happen?

The C++ standard does not specify the order in which function parameters need to be evaluated. The calling convention dictates the order in which the parameters are passed, but there’s no requirement to evaluate them the same way.

In my experience though, it’s common to see the parameters evaluated in reverse order (right-to-left). Hypothetically, a compiler may change the order if there is (for example) some opportunity for performance gain, but it seems unlikely.

> The C++17 standard defines the order of evaluation under certain circumstances. However, it will not affect the example above.
{: .prompt-info }

## How to fix it?

If you absolutely must depend on a particular order of evaluation, then explicitly do the evaluation before calling the function. Using the above example, you would call `getA()` and store the result in a local variable, then call `setB()` and store the result in another local variable. After both of those, you would call `doStuff()`, passing it those local variables.

## How to avoid it?

The example above uses console output to illustrate the problem. However, if you are careless with your coding then it’s easy to end up with something more serious. The underlying issue can often be hidden because the evaluations have side effects which you didn’t consider, such as modifying or depending on global variables.

There are various good practice guidelines which can help avoid or mitigate this issue. Avoiding reliance on global state is a good starting point (for some discussion see ["Why is Global State so evil?"](https://softwareengineering.stackexchange.com/questions/148108/why-is-global-state-so-evil)). However, you also need to be aware of other shared state, such as multiple member functions using the same member data in a class.

It can be useful to draw some lessons from [functional programming](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0) here. In theory, pure functional programming has no side effects or shared state whatsoever. That's not usually a realistic goal for an entire C++ project, but it's helpful to make elements of your code stateless where practical. Apart from anything else, it can help make your code easier to test.

The [Single Responsibility Principle](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) (the "S" from [SOLID](https://en.wikipedia.org/wiki/SOLID)) should also be considered. It's all too easy to end up with functions and classes which do too much, especially if you're doing rapid prototyping or are under time pressure. The more a unit of code does, the more likely it is to have conflicting side effects.

Lastly, remember the [KISS principle](https://simpleprogrammer.com/kiss-one-best-practice-to-rule-them-all/). Don't let your code get too complicated. The more complicated it is, the harder it is to use and maintain safely.
