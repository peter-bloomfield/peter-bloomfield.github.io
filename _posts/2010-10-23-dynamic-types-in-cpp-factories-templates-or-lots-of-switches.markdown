---
layout: post
title: 'Dynamic types in C++: factories, templates, or lots of switches?'
date: '2010-10-23 23:00:19'
tags:
- cpp
- meta-programming
---

While doing some C++ programming today, I was faced with a design decision. My code had to be able to create and delete objects of various types, dynamically and on-demand at run-time. That’s easy if you’re working with a loosely-typed language such as PHP, but it’s a different story when you’re working with C++.

The actual range of classes which could be created is fixed at compile-time. As such, there were still lots of possibilities, ranging from a simple multi-way decision, through the factory design pattern, to some interesting template metaprogramming.

## Context

First, let me explain the context. Feel free to skip this section if you’re more interested in the nuts-and-bolts of the problem and solution.

I’m working on programming my own scripting engine from the ground-up, called “avidscript”. It’s a largely C-style, strongly-typed, state-based language which gets compiled to bytecode before being executed by a stack-based virtual machine. It’s intended to be used for games, particularly for programming FSM-based AIs among other things.

As a result, it’s very important to be able to pass data between the scripts and the game. The scripting language includes the ability to define simple custom data types (much like structs in C), and I wanted a way to copy these to and from the game at run-time, without having to manually code corresponding data structures on both sides.

I’m doing this by creating the custom data types as maps at run-time in C++. Each map associates a member name to an `IDataItem` pointer, each of which can refer to some sub-class of `IDataItem`. That allows me to use dynamic polymorphism to abstract away from the actual data when handling the containing structures. When the virtual machine examines a data member coming from the script, it gets a simple identifier indicating which fundamental data type the member has.

## The Problem

The problem is fairly simple: how do I use a numeric identifier (from an enumeration) to decide which class to instantiate?

For example, let’s say we have the following enumeration:

    enum MyType
    {
     MT_TYPE_A,
     MT_TYPE_B,
     MT_TYPE_C
    };

Those identifiers correspond to the following classes:

    class CTypeA : public SomeBaseClass { ... };
    class CTypeB : public SomeBaseClass { ... };
    class CTypeC : public SomeBaseClass { ... };

If I’m given a value from `MyType`, how does my code know which class to instantiate?

## Switch statement

Here’s the very basic and very simple solution, which is entirely adequate in many cases:

    SomeBaseClass * createClass( MyType mt )
    {
        switch (mt)
        {
        case MT_TYPE_A: return new CTypeA();
        case MT_TYPE_B: return new CTypeB();
        case MT_TYPE_C: return new CTypeC();
        }
        return 0;
    }

This approach works perfectly well. There is absolutely nothing inherently wrong with it, except for the trivial overhead of the function call. There are a few other slight concerns that I would have about using it though.

Firstly, it’s a minor maintenance hazard. Every time a new class gets added, I would have to remember to modify the switch statement too.

My second concern is that I’m not instantiating and deleting the class at the same scope level. That function does the `new` bit on the class, but it’s not until some other totally unrelated bit of code that I would be doing the `delete` bit.

That’s not necessarily a problem, but when you’re dealing with a language which is not garbage collected, you need to be very careful to manage memory correctly. One way of doing that is having easily traceable new/delete pairs. Separating one or both parts of the pair into essentially unrelated sections of code is not really a good idea if you can possibly avoid it.

(Side note: In more modern C++, you'd use a smart pointer to avoid this issue.)

## Factory design pattern

The factory design pattern is a very useful one in many situations. Many games make use of it to generate elements of the game-world. For example, when a level-definition says “make bad guy X over here” and “bad guy Y over here”, it can often be some kind of factory class that actually sets-up the bad guy data and adds it to the scene.

A factory class can take many forms, but the essence of it is a class which handles the memory management and data configuration of particular parts of the program. One of the things a factory may do is store a list of all the objects it has created, and it will delete them automatically when instructed. This allows you to keep the new/delete pairs together. The basics of the class might look like this:

    class MyTypeFactory
    {
    public:
     ~MyTypeFactory() { deleteAll(); }
     SomeBaseClass * createClass( MyType mt )
     {
      SomeBaseClass * p = 0;
      switch (mt)
      {
      case MT_TYPE_A: p = new CTypeA(); break;
      case MT_TYPE_B: p = new CTypeB(); break;
      case MT_TYPE_C: p = new CTypeC(); break;
      }
      if (p) m_Objects.push_back(p);
      return p;
     }
     void deleteAll()
     {
      while (!m_Objects.empty())
      {
       delete m_Objects.front();
       m_Objects.pop_front();
      }
     }  
    private:
      std::list<SomeBaseClass*> m_Objects;
    };

That’s a very roughly thrown-together example. You would likely extend it by allowing the programmer to delete specific objects at will, rather than waiting to delete everything all at once. However, the principle is there, and it definitely has some strong uses. You would create an instance of the factory class, and then use it to create and delete all your other classes when necessary.

So what’s wrong with using a factory solution? Once again, there’s nothing inherently wrong with the idea. It can be made to work very well. The only minor downsides are that it creates a small processing/memory overhead, and it removes a little of the control of memory management.

It’s also worth noting that careless use of the factory design pattern can cause memory access violations. If your factory is expecting to delete the objects then you have to make sure you don’t delete them anywhere else. One solution to this is to make the constructor/destructor of the classes private, and make the factory a “friend” class.

## Templates

Sometimes, it’s nice just to be able to use the new/delete keywords yourself so you know exactly what’s going on with your memory allocations. Template programming provides a way to do this that avoids needing other functions or factory classes.

However, it only works if you know which classes are being created at compile-time. All the overhead exists only at compile-time, so it lets your code be elegant and efficient, albeit slightly less readable. Here’s one approach I tried:

    template < MyType T_mt > class MyTypeClass;
    template < > class MyTypeClass<MT_TYPE_A> { public: typedef CTypeA type; };
    template < > class MyTypeClass<MT_TYPE_B> { public: typedef CTypeB type; };
    template < > class MyTypeClass<MT_TYPE_C> { public: typedef CTypeC type; };

Here’s how you would use it in practice:

    SomeBaseClass *p = new MyTypeClass< MT_TYPE_C >::type(); //... delete p;

If you’re not familiar with C++ templates then it may look like a bit of a mystery. Hopefully you can still see the elegance of the solution though. If I add any new classes then maintenance is fairly minor. I just need to add a new one of those lines which starts “template \< \>”.

The actual “new” statement looks quite gruesome if you aren’t comfortable with templates, but it’s fantastically efficient. The compiler simply resolves the whole thing to a straight-up class name, and it works as though it had been typed manually.

## How the templates work

I’ll quickly explain a little of the template programming concepts, in case you’re not sure what’s going on. The first line declares a basic template class called `MyTypeClass`. The template takes a compile-time parameter of type `MyType` (that’s the stuff between the angle brackets).

Note, however, that the basic `MyTypeClass` declaration doesn’t have a body — this means it technically doesn’t exist unless we provide some explicit declarations. That’s what the lines below it are. The empty angle brackets indicate that we’re dealing with an explicit template declaration. Notice that the angle brackets have now appeared after the class name too, but this time with a value of type `MyType` — that’s the explicit declaration. This means we’ve created a version of the `MyTypeClass` for each of those `MyType` values.

Following each explicit template declaration is a simple class body containing a public `typedef`. The body can be completely different for each explicit template declaration, although in this case we need to give the typedefs the same name.

The actual leg-work happens when the compiler hits the `new` line in the usage example below that. It starts out by seeing `MyTypeClass` and the angle brackets, which tells it that it’s dealing with a templated class. It then looks for an explicit declaration which matches the `MT_TYPE_C` parameter, and if it finds it then it uses it. (If it doesn’t find it then it looks for a generic declaration, but remember we don’t have one because the first `template` line doesn’t have a class body, so the compiler would fail with an error.)

It then sees the scope operator and the word `type`. In all our explicit template declarations, the word `type` is defined as a typedef so it does a straightforward substitution for the `typedef` target. The whole mess of stuff after the `new` keyword resolves directly to `CTypeA`, `CTypeB` or `CTypeC`. The parentheses afterwards act just like the parameter list for a normal constructor.

This kind of template programming technique is quite similar to an area called “class traits”. The STL uses class traits a lot for things like string programming so that the compiler can know certain properties of different types of character that could make up a text string. Imagine replacing our `typedef` above with lots of other bits of information, or even with methods and nested-classes. The possibilities are huge.

## Conclusion

As much as the template solution is elegant and interesting, it doesn’t fully work for my purposes. It was a great little exercise, but my code will need to generate classes based on data that isn’t available until run-time (namely, the script which is running in the VM).

The crucial thing to remember about template programming is that all parameters must resolve directly to data which is fixed at run-time. In other words, you cannot use variables or any data in memory as template parameters. The chances are therefore that I will actually use a simplified factory approach, although as my work on avidscript progresses, my whole approach to this particular area may change.

<!--kg-card-end: markdown-->