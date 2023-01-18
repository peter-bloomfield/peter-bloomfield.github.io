---
layout: post
title: How to check that a function is constexpr
date: '2015-10-18 09:34:34'
tags:
- cpp
- unit-tests
redirect_from:
- /how-to-check-that-a-function-is-constexpr
---

I was implementing a number of simple [`constexpr` functions]({% post_url 2015-10-18-cpp-constexpr-functions %}) in a recent C++ project. While writing unit tests alongside it, I quickly realised that it would be very helpful to have code which verifies that a `constexpr` function can actually be evaluated at compile-time. It turns out that it’s remarkably easy to do.

## Using static assert

Static asserts are another handy feature introduced in C++11. They let you verify compile-time conditions, such as template parameters and data type sizes. Helpfully, you can call `constexpr` functions as part of their condition. Obviously, if the function cannot be evaluated at compile-time then attempting to do this will fail.

Here’s an example:

```cpp
template <typename T_ty>
constexpr T_ty pi()
{
    return T_ty(3.141592653589793);
}
static_assert(
    pi<float>() == 0 || true,
    "Compile-time evaluation check."
);
```

This is the same `pi()` function from my [previous post]({% post_url 2015-10-18-cpp-constexpr-functions %}), but this time we’ve added a `static_assert` underneath. In practice, this assertion doesn’t need to be directly underneath. It could be anywhere at all as long as it’s able to call our function.

In the event that `pi()` is not a valid `constexpr` function, an error will be reported within the `static_assert`, saying something like “expression did not evaluate to a constant”. You can test this by removing the `constexpr` from the function signature and compiling the code. Note that the assertion itself is not being triggered. Rather, an error is being reported because it’s trying (and failing) to call our function at compile-time.

The second half of the condition (`|| true`) is there to prevent the assertion from being triggered under any circumstances. This is because we’re not interested in testing _result_ of the function here. Rather, we just want to test how it gets called.

## Why do you need to check?

Normally, the compiler should give you an error message if you write code in a `constexpr` function which violates the [restrictions](http://en.cppreference.com/w/cpp/language/constexpr) on compile-time evaluation. For example, trying to call another function which isn't `constexpr`. Note that the restrictions depend on which version of the C++ standard you are compiling against.

The assert isn’t there to guard against that situation. Rather, it’s there to guard against human error by testing the assumption that it’s declared as `constexpr`. Imagine you’ve got a long-standing code base, and somewhere in the future somebody unwittingly deletes the `constexpr` keyword from the function signature. This could happen by mistake or through lack of experience, and it’s entirely possible that nobody would notice it because the compiler has silently switched to runtime evaluation. A simple static assert in your regular test code could help catch the issue.

This may sound like an unlikely scenario, but mistakes do happen, especially when people are under pressure. Sometimes it’s useful to guard against them.
