---
layout: post
title: C++11 auto variables
date: '2013-01-30 23:09:50'
tags:
- cpp
---

The C++11 standard introduced the `auto` keyword for type deduction. One of its main uses is to make variable declarations more concise. It can also improve the maintainability of code if used correctly as it can reduce redundancy. This post will give an overview of what it does and how to use it.

(Side note: Technically, C++11 [repurposed the auto keyword from C](https://www.geeksforgeeks.org/storage-classes-in-c/), but we won't cover that here.)

## A quick look

When you create and initialise a variable or object, you normally specify its type and name, and possibly some initialisation value or parameters. The `auto` keyword replaces some or all of the type declaration. For example, let’s say we’re copying a hypothetical `Widget` object like this:

    Widget foo = blah;

The compiler already knows that blah is an instance of `Widget` so telling it that `foo` is also a `Widget` is a little redundant. With the `auto` keyword, the statement can be rewritten like this:

    auto foo = blah;

This tells the compiler that `foo` is going to be the same basic type as `blah`. It’s important to realise that the actual compiled code underneath is exactly the same as the previous statement. The `foo` variable is still strongly-typed as a `Widget`. You can’t change its type later by assigning something else to it.

That’s the basic gist of `auto` in variable declarations. It’s as simple as that. You can use it for all kinds of local/global initialisations, whether you’re copying an object or a fundamental type directly, storing a literal, or storing the return value of a function.

## More realistic examples

The quick look above was deliberately very simple. Let’s look at some more realistic uses of the `auto` keyword.

### Iterators

Let’s say you have a list of `Widget` objects and you want to iterate over them. Here’s what the conventional code to get the first iterator might look like:

    std::list<Widget>::iterator iter = widgets.begin();

The type declaration is starting to become quite cumbersome. If we used the auto keyword instead, it would look like this:

    auto iter = widgets.begin();

The compiler can safely look at the declaration of the `begin()` method and determine that it returns an iterator into a list of `Widget` objects. As you can see, the code looks cleaner. If you really need to know the exact type of `iter` then modern IDEs will often show you if you just hover the mouse over the variable name.

Also, if you were to change the container type (e.g. to use a vector instead of a list) then the `auto` declaration would still work without any modification. That can make some maintenance and refactoring tasks easier and more robust.

### Smart pointers

One of the other great parts of C++11 is the robust smart pointers. Unfortunately, the declarations take up quite a few extra characters. Let’s say you’re making an instance of a `Widget` object and storing it using a shared pointer. Here’s the non-auto version:

    std::shared_ptr<Widget> p = std::make_shared<Widget>();

We can simply drop-in the auto keyword to replace the type declaration of pointer variable `p`:

    auto p = std::make_shared<Widget>();

In case you’re not familiar with it, `make_shared<>()` is the safe way to instantiate an object and immediately put it in a `shared_ptr`. You should not usually do it like this:

    auto p = std::shared_ptr<Widget>(new Widget); // not safe!

### Lambdas

My favourite part of C++11 is lambdas. These are locally-defined function objects which you can store and pass around. Normally, storing them requires the use of `std::function` which can be slightly unwieldy. This example stores a simple lamdba which just multiplies a number by 2:

    std::function<int(int)> func = [](int n) { return n * 2; };

For local storage of lambdas (especially complicated ones), `auto` declarations make life considerably easier:

    auto func = [](int n) { return n * 2; };

In case you’re wondering about the lambda’s return type, the compiler infers it automatically if the body of the lambda contains nothing but a return statement.

### const

If you want your `auto` declaration to be `const`, then you’ll normally have to specify that explicitly. Let’s say you’ve got a map of `Widget` objects and you’re trying to find one by its ID number. You might want to make the returned iterator `const` if it’s not going to be moved later, e.g.:

    const std::map<int, Widget>::iterator iter = widgetMap.find(123);

To get the same effect with auto, you need to keep the const keyword where it usually is:

    const auto iter = widgetMap.find(123);

This is a good example of how the `auto` keyword can make your code simpler. With `auto`, you don’t need to worry about the exact template arguments for the iterator declaration because the compiler can infer them directly from the `find()` method.

### References

You’ll have to add the ampersand (`&`) explicitly if you want your `auto` declaration to be a reference. For example, let’s say you have a vector of `Widget` objects and you want to get a reference to a specific element. Here’s what the conventional code might look like:

    Widget &w = vec[17];

Here’s the auto version:

    auto &w = vec[17];

If you leave out the ampersand, you’ll be copying the `Widget` instance instead of getting a reference to it.

You can explicitly specify `const` as well if you want to. However, note that if the right-hand side of the assignment is already `const` then your `auto` reference will be implicitly `const` as well. For example:

    const int num = 10;
    auto & ref = num; // ref is a const int &

## Where not to use auto

### Members and parameters

In C++11, the `auto` keyword only works for local and global variables. That means you cannot use `auto` to declare a class member or a function parameter. This is because the declaration of a structure or function must be explicit and unambiguous.

If you want something similar to `auto` behaviour for member variables and function parameters then you might find templates useful.

(Note: C++14 added the ability to declare `auto` parameters in lambdas.)

### Implicit up-casts

Despite being generally considered a strongly-typed language, C++ is a little lax about some conversions. For example, you can implicitly up-cast a pointer from a derived class to a publicly-inherited base class:

    ParentClass * ptr = new ChildClass;

If you use auto carelessly here then you actually change the meaning of the code:

    auto ptr = new ChildClass;

`ptr` is now a pointer to a `ChildClass` object. Depending on your class design, this may or may not be a problem. In many cases, it will be completely harmless. However, it’s important to be aware of implicit conversions that you may be relying on before using auto. An explicit declaration would probably be most appropriate for cases like this, but you could also cast the right-hand side of the assignment.

### Implicit literal conversions

This is a similar issue to implicit up-casts, and it particularly affects integer types due to type promotion. The following example stores an integer literal as an unsigned short (typically a 2 byte type):

    unsigned short num = 123;

If we use `auto` here, we’ll again have unintended behaviour:

    auto num = 123;

The `auto` declaration will resolve to signed int (typically a 4 byte type) because that’s how the compiler will usually interpret an integer literal. Once again, an explicit declaration is typically the best way to go here to avoid the ambiguity.

## Common concerns

### Strong vs. weak types

Some people might look at `auto` variables and think C++ is turning into a weakly-typed language. That’s absolutely not the case because the type behaviour is completely unmodified. A variable declared as `auto` simply has its type deduced (primarily) by the right-hand side of the initialisation statement instead of the left-hand side. That’s all there is to it. It’s still as strongly-typed and as type-safe as ever.

### Performance

Another common concern is that auto variables will somehow affect the performance of the program. As with strong vs. weak types this is a non-issue because the type behaviour is completely unmodified. The only difference with auto is where the compiler looks to determine the declaration type. The run-time behaviour and performance should be completely unaffected (unless you’re careless about how you introduce `auto` into your code).

## Conclusion

The `auto` keyword is a very handy little addition to the arsenal of day-to-day programming tools. I’ve only covered one aspect of its use here, but it’s the aspect which most programmers will probably see and use most often.

It’s vital to remember though that it doesn’t change the underlying code at all. It’s simply a bit of ‘syntactic sugar’ to make your life a little easier.

<!--kg-card-end: markdown-->