---
layout: post
title: MFC prevents bad_alloc from being thrown
date: '2015-04-10 19:05:12'
tags:
- cpp
redirect_from:
- /mfc-prevents-bad_alloc-from-being-thrown
- /mfc-prevents-bad_alloc-from-being-thrown/
---

According to the C++ standard, the `new` operator should throw `std::bad_alloc` if it fails. This will typically happen if your process has run out of memory. However, this isn’t the case if your program uses the (rather outdated) [Microsoft Foundation Class](https://docs.microsoft.com/en-us/cpp/mfc/mfc-desktop-applications) library. In this post, we’ll look at what’s going on, and what you can do about it.

## The normal approach

In practice, catching `bad_alloc` isn’t usually very useful. If your program has truly run out of available memory then there’s often very little you can do to salvage the situation. However, catching it can occasionally be useful if your program needs an unusually large chunk of contiguous memory. In these cases, you might detect a failed allocation like this:

```cpp
char *buffer = nullptr;
try
{
    // Attempt to allocate a large block of memory
    buffer = new char[10000000];
}
catch (const std::bad_alloc &)
{
    // Allocation failed. Attempt to recover here...
}
```

However, if your program links against the MFC libraries, then the `bad_alloc` handler will never be triggered.

## What happens instead?

Without any MFC-aware exception handling, your program will probably display a simple little dialog saying “Out of Memory”, and that’s it. It might seem to continue OK, or it might eventually crash or have other problems, depending on how you’ve written it. It may not say anything at all about an unhandled exception because MFC seems to absorb it (sometimes).

As far as I can tell, MFC replaces the standard `new` operator with its own version. When it fails, instead of throwing `bad_alloc`, it will throw a **pointer** to a `CMemoryException` object.

The pointer is important. It means you can’t catch it by value or reference.

This means you need to detect a failed allocation something like this instead:

```cpp
char *buffer = nullptr;
try
{
    // Attempt to allocate a large block of memory
    buffer = new char[10000000];
}
catch (const CMemoryException *)
{
    // Allocation failed. Attempt to recover here...
}
```

Standard MFC exception handling actually looks a little different. It uses its own macros such as `CATCH()`. These disguise the fact that the type being thrown/caught is actually a pointer. Using macros this way is generally bad practice by modern standards as it obscures the underlying code. It's not a technique which should be copied.

(In fact, modern C++ should avoid the use of macros altogether where possible. There are some situations where they are the right tool for the job, but they should be used very sparingly.)

## Why does this happen?

MFC is simply very out-dated. Throwing `bad_alloc` wasn’t standard back when MFC was first introduced, and changing its behaviour now could cause problems for existing code. Even though MFC is still maintained (to a certain extent), I suspect they will never fix it because they don’t really seem to want people using it anymore.

## Portable solutions

Unfortunately, you can’t catch an incomplete type in C++. You can't even catch a pointer to one, so a forward declaration doesn't work. Everything which needs to catch `CMemoryException` needs to see its full declaration, creating a frustrating dependence on the MFC headers.

That’s not a problem if it’s within code which is tied to MFC anyway, such as a dialog’s event handler. However, it’s not good for code which needs to remain portable, such as a class which you might reuse in another application.

### Avoid the new operator

Since the problem is that the `new` operator is being replaced, you could simply try to avoid using it. This isn't very practical for any significant program, but it might be feasible in limited situations.

It's also possible that you could use smart pointers and standard library containers instead, such as `std::shared_ptr` and `std::vector`. These don't always use the `new` operator internally for a variety of reasons. However, it does depend on a few factors so it may be risky to rely on.

### Catch everything

A more reliable alternative would be to use a catch-all around potentially risky allocations:

```cpp
char *buffer = nullptr;
try
{
    // Attempt to allocate a large block of memory
    buffer = new char[10000000];
}
catch (...)
{
    // Allocation failed. Attempt to recover here...
}
```

That will catch any exception which is thrown. If the only code within the `try` block is an allocation of Plain Old Data (e.g. `char` or `int`) then this is probably safe. `CMemoryException` and `bad_alloc` should be the only exceptions you’ll see.

However, in pretty much all other circumstances, there is a risk of hiding some other exception which may be important. As such, you need to be careful with this approach.

### typedef

A reasonably painless approach which I’ve used successfully is to `typedef` the exception type. In a header file somewhere (probably `stdafx.h` if you’re using it), create a `typedef` like this:

```cpp
typedef const CMemoryException * FailedAllocException;
```

This allows you to create exception handlers like this:

```cpp
char *buffer = nullptr;
try
{
    // Attempt to allocate a large block of memory
    buffer = new char[10000000];
}
catch (FailedAllocException)
{
    // Allocation failed. Attempt to recover here...
}
```

If you want to use the code in a non-MFC environment, simply change the `typedef` to something like this:

```cpp
typedef const std::bad_alloc & FailedAllocException;
```

It's important to note that the `CMemoryException` instance should technically be freed otherwise you will have a memory leak. This could perhaps be done with a conditional macro.

### nothrow

In theory, the [`nothrow`](https://en.cppreference.com/w/cpp/memory/new/nothrow) overload of the `new` operator should be a viable alternative:

```cpp
char *buffer = new (std::nothrow) char[10000000];
```

This is supposed to avoid throwing exceptions entirely. If something goes wrong, it will instead return a null pointer. Unfortunately, MFC seems to break this as well. I couldn’t get it to work on VC12 anyway.

### Replace the new operator or handler

A more radical solution could involve replacing the `new` operator, or replacing the handler function which is called when `new` fails (calling [`set_new_handler()`](https://en.cppreference.com/w/cpp/memory/new/set_new_handler)). This generally isn't a good idea though as it could break other code which may be relying on specific/standard behaviour.

## Conclusion

As already mentioned, the MFC library is very outdated, having first been released in 1992. That was several years before the core of the C++ language was standardised. As a result, its design doesn't fit well with modern code, despite having been an impressive feat of software engineering in its day.

The difficulty with the `new` operator is just one of many issues arising from its age. MFC is technically still under development by Microsoft, although I doubt it will ever see a major overhaul.

Consequently, I strongly advise against using it in any new project. Where possible, older projects should be migrated to another GUI library, or possibly to another language such as C# if appropriate.
