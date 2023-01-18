---
layout: post
title: C++ constexpr functions
date: '2015-10-18 08:51:46'
tags:
- cpp
redirect_from:
- /cpp-constexpr-functions
---

I’ve been very excited to see that Visual Studio 2015 supports the `constexpr` keyword in C++. It was introduced in the C++11 standard, and is being taken further in upcoming revisions.

There are a number of uses for the keyword, but the one which excites me the most is using it for writing functions which can be executed during compilation, potentially saving a lot of runtime overhead. In this post, I’ll show a couple of quick examples.

## Maths constants

What do you do when your program needs to use a maths constant, such as pi? One traditional approach is to use a macro like this:

```cpp
#define PI 3.141592653589793
```

One of the major problems with a macro is that it isn’t type-safe. There are various ways to solve this, but here’s an approach which uses a `constexpr` function:

```cpp
template <typename T_ty>
constexpr T_ty pi()
{
    return T_ty(3.141592653589793);
}
```

To use this, you’d simply call the `pi()` function with your desired type as the template parameter. For example:

```cpp
float twoPi = 2.0f * pi<float>();
```

The nice thing about this is that there’s no runtime overhead from a function call or type conversion. The `pi<float>()` call is performed during compilation and the expression is effectively replaced by a literal value (i.e. the result of the call).

## Adding function parameters

A common maths utility with games programming is converting angles between degrees and radians. We can write a `constexpr` function to do that too:

```cpp
template <typename T_ty>
constexpr T_ty degToRad(T_ty angle)
{
    return angle * (pi<T_ty>() / T_ty(180));
}
```

To use this function, you’d call it something like this:

```cpp
float angle = degToRad(47.0f);
```

The compiler can infer the return type from the parameter you pass in, which is why the explicit template parameter isn’t necessary.

## Runtime vs. compile time

In the `degToRad()` example above, it can only be executed at compile time if it’s passed a literal or `constexpr` value; i.e. something whose value is known fixed at compile-time.

But what happens if you pass a run-time variable to the function? For example:

```cpp
float deg = 0.0f;
cin >> deg;
float rad = degToRad(deg);
```

This takes the input angle from stdin at runtime so there’s absolutely no way its value can be known at compile time. In this case, the compiler automatically invokes the function at runtime instead.

This is a very handy feature of `constexpr` functions: you only need to write it once, but you get compile-time and runtime versions to play with. The compiler automatically figures out how to call it.

## Recursion

Notice that our `degToRad()` function above is calling our earlier `pi()` function. The ability to do this means you can offload some surprisingly intensive calculations to the compiler. Obviously this could slow down your build times, but will potentially improve execution times.

A good example of this is a recursive function call. Here’s an example which will calculate pi to any positive integer power, as long as the compiler doesn’t run out of memory:

```cpp
template <typename T_ty>
constexpr T_ty piRaised(unsigned int power)
{
    return (power == 0) ?
        T_ty(1) : // < Terminal case (anything to 0th power is 1)
        pi<T_ty>() * piRaised<T_ty>(power - 1); // < Recursion
}
```

As long as it’s called with a literal or `constexpr` value, this function’s entire recursion will happen during compilation.

## Restrictions

C++11 has some fairly serious restrictions on what can go into a `constexpr` function. The executable code is limited to a single return statement and nothing else. You can have static asserts and typedefs, but not much else. If you work with VS2015 or similar era compiler then you’ll be limited to this.

C++14 relaxes the restrictions somewhat, allowing multiple statements among other things. More information can be found [here](http://en.cppreference.com/w/cpp/language/constexpr).

## Conclusion

I’ve shown a couple of really simple examples of what you can do with `constexpr` functions. Bearing in mind that they can call each other, they really open up a whole world of helpful optimisations if used carefully. What’s more, it’s even possible to have `constexpr` member functions and constructors, so there’s lots more to explore here beyond what I’ve demonstrated.

It's also worth mentioning that some of the standard library functions are declared as `constexpr`, depending on which version of the C++ standard you're using. For example, see [`std::min()`](https://en.cppreference.com/w/cpp/algorithm/min) and [`std::max()`](https://en.cppreference.com/w/cpp/algorithm/max) from C++14 onwards.
