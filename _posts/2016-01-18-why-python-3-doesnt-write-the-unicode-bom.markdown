---
layout: post
title: Why Python 3 doesn't write the Unicode BOM
date: '2016-01-18 18:15:45'
tags:
- python
- unicode
redirect_from:
- /why-python-3-doesnt-write-the-unicode-bom
---

I’ve been using Python scripts to automatically edit and output Windows Resource files (.rc) for C++ projects in Visual Studio 2013. When handling Unicode, Windows and Visual Studio always want little endian UTF-16 encoding, and the resource file should always start with the Unicode BOM (Byte Order Mark). However, despite the promises in the documentation, I found that Python wasn’t outputting the BOM automatically.

I nearly resorted to outputting it manually, but as is often the case with Python, the correct approach is simpler than it seems.

## What is the Unicode BOM?

The Unicode BOM (Byte Order Mark) is a character which can occur at the start of a Unicode text file to indicate what [endianness](https://en.wikipedia.org/wiki/Endianness) the data is stored in. It’s very helpful for portability as it means programs on different systems can automatically detect the encoding. This allows them to display, edit, and store the text appropriately, leaving no room for ambiguity.

Endianness is only relevant for encodings which use more than one byte per code unit, such as UTF-16 and UTF-32. There is also a BOM for UTF-8. However, it’s only used to identify the file as being UTF-8, as opposed to ASCII or some other encoding. Byte order is irrelevant in that case, and the UTF-8 BOM is actively discouraged.

## Python and BOM

According to the Python documentation on [reading and writing Unicode data](https://docs.python.org/3/howto/unicode.html#reading-and-writing-unicode-data):

> Some encodings, such as UTF-16, expect a BOM to be present at the start of a file; when such an encoding is used, the BOM will be automatically written as the first character and will be silently dropped when the file is read.

From this, it sounds like any UTF-16 or UTF-32 encoding will automatically take of the BOM. However, try running the following code in a Python 3 script:

```python
with open("output.txt", mode="w", encoding="utf-16-le") as f:
    f.write("Hello World.")
```

Open the output file in an editor which reports the encoding, such as [Notepad++](https://notepad-plus-plus.org/) on Windows. You’ll see UTF-16 (or UCS-2) Little Endian, but it will say there is no BOM.

## Where did the BOM go?

To answer that, look at the encoding argument in the code snippet above. It’s set to `utf-16-le`, which explicitly indicates Little Endian encoding. It turns out that **if you explicitly specify endianness, Python assumes you don’t need a BOM**. This is actually mentioned in the official documentation, but not particularly clearly.

Instead, change the encoding to `utf-16`. This lets Python use the Operating System’s endianness, and it assumes that a BOM is therefore necessary. Here’s the modified code snippet:

```python
with open("output.txt", mode="w", encoding="utf-16") as f:
    f.write("Hello World.")
```

Once again, run that as a Python 3 script and then open the output file. Assuming you’re running on a little endian system (which should apply to anything running a Windows OS), the encoding should show up as UTF-16 (or UCS-2) Little Endian with BOM.
