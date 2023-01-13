---
layout: post
title: Using C++ templates for size-based type selection
date: '2015-10-12 18:36:02'
tags:
- cpp
- meta-programming
- templates
---

The standardisation of size-specific integer types in C/C++ is extremely useful for portability. That is, when you use types like `uint16_t` and `int32_t`, you know exactly what size of data type you’re getting (assuming your compiler supports it). This isn’t the case with the more traditional types like `short` and `int` whose sizes can vary from one compiler to another.

However, what happens if you want to write templated code which automatically selects an integer size based on a template parameter? This post outlines one possible solution using [`std::conditional`](https://en.cppreference.com/w/cpp/types/conditional) which was introduced in C++11.

## Implementation

This makes use of a `using` declaration, which is like a templated version of `typedef`. Within it are three nested conditional templates:

    template <std::uint8_t T_numBytes>
    using UintSelector =
        typename std::conditional<T_numBytes == 1, std::uint8_t,
            typename std::conditional<T_numBytes == 2, std::uint16_t,
                typename std::conditional<T_numBytes == 3 || T_numBytes == 4, std::uint32_t,
                    std::uint64_t
                >::type
            >::type
        >::type;

Example usage:

    UintSelector<6> var = 185;

A more complete example is available [here on ideone](http://ideone.com/UAK86I).

The relationships between sizes and types is:

- 1 byte =\> std::uint8\_t
- 2 bytes =\> std::uint16\_t
- 3-4 bytes =\> std::uint32\_t
- 5+ bytes =\> std::uint64\_t

One limitation to note is that this only goes up to 8 bytes (64 bits). Anything higher will still give you `std::uint64_t`. If you want to handle larger sizes then you’ll need some other approach, such as using an array instead of a scalar type. Alternatively, you may want the compiler to emit an error if anything larger is attempted.

## How does std::conditional work?

The workhorse of the code is the conditional template. It was introduced in C++11, although it was certainly possible to implement it manually before that. The form is as follows:

    std::conditional<bool condition, typename ifTrue, typename ifFalse>

The conditional class has a typedef inside named `type`. If the condition is true then the typedef is defined as the `ifTrue` template parameter. Otherwise, it is defined as `ifFalse`. (These are arbitrary names which aren’t specified in the standard.) In the underlying code, this could be done through partial template specialisation.

As in the example above, the conditionals can be nested. If the condition is true then it resolves to a specific type. Otherwise, it resolves to another conditional, and so on until you reach the final conditional in the chain.

Note that you have must provide a valid `ifFalse` template parameter. If you only want `ifTrue` then you might want to look into [std::enable\_if](http://en.cppreference.com/w/cpp/types/enable_if) instead.

One important thing to be aware of in the code above is the need to prefix the conditional with `typename`. Without it, you’ll get a compiler error saying something like “dependent name is not a type”. [This Stack Overflow question](http://stackoverflow.com/questions/610245/where-and-why-do-i-have-to-put-the-template-and-typename-keywords) has some excellent information about this issue in the answers.

## What’s it good for?

I originally explored this while tinkering with ideas for a maths library. I’d been looking at implementing a runtime-resizeable equivalent of [`std::bitset`](https://en.cppreference.com/w/cpp/utility/bitset), which would form the basis for arbitrary sized integers and fixed point types. The idea is that the data would be split up into chunks, allowing various binary operations to happen chunk-at-a-time instead of having to iterate over every individual bit.

Hypothetically, a programmer using one of the classes would be able to specify the chunk size that would be the fastest computationally, and/or the least wasteful in terms of memory. In practice, I’m not sure it would make a lot of difference though. The increased complexity of the code (and the increased likelihood of bugs) might outweigh the marginal performance benefits.

There are other places this could come in handy too, such as selecting an integral type whose size most closely resembles another type (such as a class). Perhaps this could be useful in serialization for network transmission.

## Other approaches

In most cases, it would make more sense just to specify the desired type directly in code or as a template parameter. The only real benefit I can think of for size-based selection is ensuring a particular category of type is used, such as signed vs. unsigned. You can currently use [type traits](http://en.cppreference.com/w/cpp/header/type_traits) and [static asserts](http://en.cppreference.com/w/cpp/language/static_assert) to guard against that though, and hopefully we might see [concepts](https://en.wikipedia.org/wiki/Concepts_(C%2B%2B)) make a come-back in a future version of the standard.

<!--kg-card-end: markdown-->