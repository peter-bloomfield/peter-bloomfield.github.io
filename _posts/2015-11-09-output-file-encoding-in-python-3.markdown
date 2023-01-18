---
layout: post
title: Output file encoding in Python 3
date: '2015-11-09 18:39:06'
tags:
- python
- unicode
redirect_from:
- /output-file-encoding-in-python-3
---

[Unicode](https://en.wikipedia.org/wiki/Unicode) is very widespread now (for good reason), and one of the great benefits of [Python](https://www.python.org/) 3.x is that it handles Unicode natively. There are different ways to represent Unicode though, so how do you set the file encoding in Python when you’re writing out to file?

It’s not immediately obvious from some of the official documentation, but thankfully it’s not complicated. I’ll give a quick overview in this post.

## open() function

When you’re opening a text file for input or output you’ll typically use the [open()](https://docs.python.org/3/library/functions.html#open) function. It returns a file object. Here’s the function signature:

```python
open(file, mode="r", buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

The 4th parameter specifies the encoding. By default, it’s set to `None` which means it will use the system’s preferred encoding. Alternatively, you can pass it a string naming any of the encodings supported by Python, and that’s what it will use to read or write the text. For example, to open a file as UTF-8, you could do this:

```python
f = open("hello.txt", "w", encoding="utf-8") f.write("Привет мир")
```

That opens a file called `hello.txt` and writes something like “Hello world” in Russian (I used Google translate so I’m sorry if it means something else!). For that to work properly in a script, make sure the script itself is saved with UTF-8 encoding.

As a side note, you can see that I’m using a named argument to skip the 3rd parameter of the `open()` function. This is a helpful feature of Python to be aware of.

## Available encodings

The encoding argument for `open()` can be any of the following:

- `utf-8`
- `utf-16`
- `utf-32`
- `utf-16-be`
- `utf-16-le`
- `utf-32-be`
- `utf-32-le`

The `be` and `le` suffixes stand for "big endian" and "little endian". This determines the way a system stores units of data which are bigger than one byte (see the [endianness article on Wikipedia](https://en.wikipedia.org/wiki/Endianness) for more information). If you select the `utf-16` or `utf-32` encoding _without_ an endian suffix then Python will use the system’s native endianness.

## Which encoding to use?

Generally speaking, UTF-8 seems to be the most widely used and the most portable choice, plus it doesn’t care about endianness (one less problem to worry about!). I would recommend against using UTF-16 or UTF-32 unless you have a specific reason, e.g. for interacting with another program which requires it.

## Further reading

- [Python 3 – Unicode HOWTO](https://docs.python.org/3/howto/unicode.html)
- [What Every Programmer Absolutely, Positively Needs To Know About Encodings And Character Sets To Work With Text](http://kunststube.net/encoding)
- [Unicode FAQ](http://www.unicode.org/faq)
