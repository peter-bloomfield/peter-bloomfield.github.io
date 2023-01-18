---
layout: post
title: Convert a number to a binary string (and back) in C++
date: '2015-02-19 20:04:18'
tags:
- cpp
redirect_from:
- /convert-a-number-to-a-binary-string-and-back-in-cpp
---

Sometimes it’s useful to output the binary representation of a number in text, i.e. as an ASCII string of 0’s and 1’s. There are also situations where you might want convert back the other way, e.g. if you want to let a user enter a binary string manually. The [bitset](http://www.cplusplus.com/reference/bitset/bitset/) class in C++ makes this surprisingly quick and easy.

## Number to binary string

The first thing to do is include the `<bitset>` header in your code. In the examples below, we’re outputting/storing the number 123 as an 8-bit value.

Output directly to the console (or other output stream):

```cpp
std::cout << std::bitset<8>(123);
```

Store in a string object:

```cpp
std::string str = std::bitset<8>(123).to_string();
```

The template parameter (the number between &lt;angle brackets&gt;) specifies the number of bits you want to use. You have to make sure it’s big enough to hold the number you pass into the constructor. However, if it's too big then you will end up with a lot of leading 0's in the resulting string.

It’s also important to note that the number you convert to a binary string has a finite limit. It should be able to handle at least a 32-bit unsigned integer according to the standard though (a little over 4 billion).

## Binary string to number

Going back the other way (from a binary string to a number) is just as easy:

```cpp
std::cout << std::bitset<8>("11011011").to_ulong();
```

That will output the number 219 directly to the console. Alternatively, you can also store the result in a variable:

```cpp
unsigned long n = std::bitset<8>("11011011").to_ulong();
```

In both cases, you could also pass in a string object rather than a string lteral.

Once again, these examples are using an 8-bit number. A notable limitation here is that the number of bits is a compile-time value. That means you can’t determine the length of the string at run-time and use that number of bits. You have to determine in advance how many you’re likely to need.

### Binary literals in C++14

The above assumes your binary string isn't known until runtime, e.g. because it's being entered by a user or read from a file. If you know the binary string at compile time, and you're using C++14 or later, then you don't need to use `bitset` at all. You can write a [binary literal](https://en.cppreference.com/w/cpp/language/integer_literal) directly in your code like this:

```cpp
int n = 0b10011101;
```

This is exactly equivalent to writing the number in decimal like this:

```cpp
int n = 157;
```

## Explanation

Converting numbers between binary and decimal representations isn't really the purpose of the bitset class. It's designed to represent a tightly packed binary field where specific bits may be turned on or off like boolean flags. This is helpful for storing things like configuration information.

It’s quite common to store this information as plain old integer values, using a combination of [bitwise operators](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/) to adjust the individual bits. This approach can be seen in things like the Win32 API, and is still very commonly used in C programming, especially for embedded systems. It’s not always intuitive though, and can be tricky to follow if you’re not used to it. The bitset class is designed to make this easier.

As far as I know, the string functionality we're using in the examples above was only included for convenience. I imagine they're mainly used to help with debugging and serialisation. They're not intended as the main way of interacting with bitsets, but they can certainly come in handy.
