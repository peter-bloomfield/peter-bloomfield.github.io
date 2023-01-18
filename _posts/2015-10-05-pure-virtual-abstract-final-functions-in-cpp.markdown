---
layout: post
title: Pure virtual final functions in C++
date: '2015-10-05 18:06:24'
tags:
- cpp
redirect_from:
- /pure-virtual-abstract-final-functions-in-cpp
---

Today I ran across an interesting little quirk of C++11. You can declare a pure virtual function which has no implementation and which is `final`. That means the class can never be instantiated or inherited, and the function will never have a body.

For example:

```cpp
class Widget
{
    virtual void foo() final = 0;
};
```

It certainly looks odd at first glance, but it’s legal in C++, and seems to behave correctly on recent compilers. Admittedly, it’s completely useless for most purposes. However, there’s a similar pattern which I have found useful:

```cpp
class Widget final
{
    virtual ~Widget() = 0;
};
```

This is practically the same as the first example, but is perhaps a little more readable. It’s the class rather than the function which is declared as final, so it’s easier to spot that specifier and understand what it means. Also, I’ve used the destructor instead of some other arbitrary function so that there’s no chance of naming conflicts.

The only potential use I’ve found for this is in template metaprogramming. Sometimes classes are declared solely for compile-time traits information, or to wrap a static function which needs partial specialization. In these cases, you may never want to instantiate or inherit the class.

There’s not necessarily any reason to *prevent* it as such. However, it at least means that if it happens by mistake then it will be noticed right away.
