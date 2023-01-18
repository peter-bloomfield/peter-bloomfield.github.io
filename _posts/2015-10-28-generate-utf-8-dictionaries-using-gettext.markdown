---
layout: post
title: Generate UTF-8 dictionaries using gettext
date: '2015-10-28 18:35:11'
tags:
- localisation
- unicode
redirect_from:
- /generate-utf-8-dictionaries-using-gettext
---

I’ve been setting up localisation for an application using a combination of tools: the [Boost Locale](http://www.boost.org/doc/libs/release/libs/locale) library and the widely used [gettext](https://www.gnu.org/software/gettext/) tools. I’m wanting to work entirely in UTF-8 because it should be suitable for pretty much any language we’re likely to need, and it should hopefully avoid problems of mixing encodings. However, I found that the tools kept falling-back to outdated character sets instead.

Fortunately, there’s a fairly easy solution which I’ll outline quickly in this post.

## The problem

If the string literals in your source code only contain ASCII characters, then when xgettext extracts them it won’t settle on any specific character set. When it outputs a dictionary template (\*.pot) file, it will specify the charset simply as “CHARSET” (a placeholder). If you pass the unedited \*.pot file into `msginit` or `msgmerge` (to generate a \*.po dictionary file), it will try to guess a suitable charset for you. Unfortunately, its guesswork doesn’t seem to know about Unicode so it won’t select UTF-8.

You could manually edit the charset specified in the \*.pot file. You could also alter it in the resulting \*.po files. However, to avoid mistakes, I wanted to ensure it is clearly declared as UTF-8 throughout.

## The solution

There are two things you need to do. First, you need to specify the following in the `xgettext` command line option:

```
--from-code=UTF-8
```

This tells the program what encoding it should expect to see when extracting string literals. Unfortunately though, it’s not enough on its own. If all the characters encountered are ASCII then it basically seems to ignore your UTF-8 instruction.

The second thing you need to do is ensure that xgettext extracts at least one UTF-8 character in a string literal. To guarantee this, I added a fake source code file which only contained a translation string. This file is included in the list parsed by xgettext, but it’s not actually compiled into our software.

Here’s the contents of the file, which obviously needs to be encoded as UTF-8:

```
// TRANSLATOR Ignore this entry.
translate(" __IGNORE__", "ø");
```

I’m working with C++, although the language doesn’t matter too much in this case since the code is deliberately invalid. (The compiler wouldn’t be happy about the fact that there’s a statement outside a function, although `xgettext` doesn’t care.)

As I mentioned above, I’m using the Boost Locale functionality at runtime for message translation. As such, I’ve setup `xgettext` to look for “translate” as a keyword, and to extract comments tagged with “TRANSLATOR”. This means the resulting dictionary file will include an instruction to ignore the entry. “ **IGNORE** ” is the context string which I added to ensure it doesn’t conflict with any other entries. The actual translation string (“ø”) is just an arbitrary Unicode character.

After running `xgettext`, the new entry in the \*.pot file is as follows:

```
#. TRANSLATOR Ignore this entry.
#: C:\example\force-utf-8.cpp:2
msgctxt " __IGNORE__"
msgid "ø"
msgstr ""
```

In the header data, the charset is now specified as UTF-8. This is successfully carried over into the \*.po files by `msginit` and `msgmerge`.
