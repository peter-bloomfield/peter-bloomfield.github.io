---
layout: post
title: The joy of C++ streams
date: '2011-01-08 02:15:32'
tags:
- cpp
redirect_from:
- /the-joy-of-cpp-streams
- /the-joy-of-cpp-streams/
---

Programmers sometimes complain about how poorly C++ handles strings. The unfortunate reality is that a language with such direct memory access as C++ can’t realistically handle variable-length strings natively. This is because stack memory usage absolutely must be fixed at compile-time. Or, in simpler terms, you can’t expect to fit an unpredictably large peg into a small hole.

The `std::string` class improves things considerably. It's a very robust and efficient string class which is about as close to a native string as you’re likely to get. It still doesn’t solve all our problems though. For example, it doesn’t always do string concatenation very well, and it’s not always easy to convert other types to a `std::string` (this has changed a little in more recent versions of the C++ standard).

Thankfully, streams come to our rescue! They provide a beautifully simple and straightforward means of building up strings of text and sending them pretty much wherever we need them.

## Console streams

Everybody who uses C++ ought to know about streams. They are an essential part of the C++ experience. Sadly, very few people really know much about them, even though nearly everybody has used them. Consider the following code:

```cpp
int num = 17;
std::cout << "Hello world! My number is " << num << std::endl;
```

In a console app, it displays on the screen: `Hello world! My number is 17`. Pretty much everybody has done something like that at some stage when learning C++. But did you know that `cout` is in fact an output stream which just happens to write to the console? Stream data can be diverted anywhere you like.

## File streams

I’ve seen people struggle terribly with C-style `fopen(..)`, `fread(..)`, and `fwrite(..)` functions when doing file I/O. There's no need because you can use file streams just like you used the console stream. Here’s our console example adapted for file output:

```cpp
int num = 17;
std::fstream file("test.txt", std::fstream::out);
file << "Hello world! My number is " << num << std::endl;
file.close();
```

That should give you a file named “test.txt” with the same text in it as we had on our console earlier.

## String streams

Now for the really fun part. You can make a stream just store up the data you give it, then get the whole lot out as a `std::string`. No need for allocating char buffers and using C-style `sprintf()` anymore. Here’s how:

```cpp
int num = 17;
std::stringstream stream;
stream << "Hello world! My number is " << num << std::endl;
std::string s = stream.str();
```

It’s really that easy. You just need to remember to call `.str()` on your stream object to get a conventional `std::string` object out. You can use string streams to write data of many kinds into a string. You can also parse data out of a string too, like this:

```cpp
float f = 0.0f;
int i = 0;
std::stringstream("123.4") >> f;
std::stringstream("321") >> i;
```

At the end of that, your float will contain `123.4` and your int will contain `321`. That kind of data extraction/conversion works for any stream, including the console and files.

## Manipulators

The fun of C++ streams doesn’t end there. There are wonderful things called manipulators which can modify your input/output. `std::endl` is an example of a manipulator which ends a line and flushes the buffer. Here’s another very handy one for programmers:

```cpp
std::cout << std::hex << 29;
```

That will output `1d` to the console. The `hex` manipulator will modify the stream so that every number you subsequently send down it will be expressed in hexadecimal. You can also use `oct` for octal, or `dec` to switch it back to decimal. There are also manipulators to express numbers in fixed-point or scientific notation, as well as many more.

## More information…

I heartily recommend you take a look at all that’s on offer with C++ streams, because they can save you a lot of time and effort. If only somebody had told me about them when I was learning!

I find [cppreference.com](https://en.cppreference.com) to be an invaluable resource on this and other topics. Here are some articles to get you started:

- [`iostream`](https://en.cppreference.com/w/cpp/header/iostream) – for console input/output
- [`fstream`](https://en.cppreference.com/w/cpp/header/fstream) – for file input/output
- [`sstream`](https://en.cppreference.com/w/cpp/header/sstream) – for string reading/writing
- [`manipulators`](https://en.cppreference.com/w/cpp/io/manip)
