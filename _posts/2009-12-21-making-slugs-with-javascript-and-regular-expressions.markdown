---
layout: post
title: Making slugs with JavaScript and regular expressions
date: '2009-12-21 15:05:42'
tags:
- javascript
- web-dev
---

You will sometimes want to generate a suitable URL ‘slug’ based on something the user has entered, such as the name of a blog post, or perhaps the name of a file which is being uploaded.

In order to ensure that your URLs are consistent, valid, and unambiguous, it is common to restrict the slug to ASCII letters (lower-case only), numbers, and dashes (-). For example, a blog post called “Hello World!” might have a resulting slug of “hello-world”. Additionally, it is common to remove leading and trailing dashes from the final slug.

In this post, I’ll introduce a way of doing this in pure JavaScript.

## Why JavaScript?

JavaScript is the standard client-side scripting language on the web, enabling all sorts of dynamic and interactive behaviour. For example, it might be useful for a form to automatically generate a slug based on something the user is typing. For WordPress does it when you type the name of a new blog post.

## How?

First, convert the string to lowercase:

    str = str.toLowerCase();

We could then use a regular expression to convert all invalid characters into dashes:

    str = str.replace(/[^a-z0-9\-]/g, '-');

That line might look a little odd if you are not familiar with regular expressions (see the W3Schools JavaScript reference on the [RegExp](https://w3schools.com/jsref/jsref_obj_regexp.asp) object for full details). The part in square brackets says “find any character which is _not_ in ranges a-z or 0-9, and which isn’t a dash”. The /g tells it to find all matches (not just the first one). And the ‘-‘ at the end tells it to replace each of these matching characters with a dash (-).

That regular expression is quite effective, but if you try putting in “Hello !-# World” then the output slug will be “hello—–world”. This is because each invalid character was replaced by a dash one-by-one, resulting in multiple consecutive dashes. This is not ideal, so we can refine our regular expression to look like this:

    str = str.replace(/[^a-z0-9]+/g, '-');

The difference is subtle. Firstly, we’ve taken the dash out of the square brackets. This means that all dashes will be replaced too. Secondly, we’ve put a plus sign after the square brackets. This means it will no longer match one character at a time. Instead, it will match groups of 1 or more consecutive characters, and replace the whole group with a single dash. That means our “Hello !-# World” example becomes simply “hello-world”.

We’re not quite done yet though, because we haven’t taken leading and trailing dashes into account. If we put “!Hello World!” into our code so far, we’d end up with a slug saying “-hello-world-“. The leading and trailing dashes are unsightly so how do we get rid of them?

The simplest and most readable way is probably to use string functions to check/replace the first and last characters explicitly. However, a cleaner and possibly more efficient way would be to use a second regular expression replace. This is in addition to the one above:

    str = str.replace(/^-+|-+$/g, '');

Once again, regular expressions can be difficult to understand if you are new to them. Going from left to right, here’s what it means:

- **^-+** = Match a group of one or more consecutive dashes at the beginning of the line.
- **|** = Match either what’s on the left or on the right of this character.
- **-+$** = Match a group of one or more consecutive dashes at the end of the line.
- **g** = Replace all instances in the string.

It is important to realise that this means a slug consisting only of invalid characters or dashes will be completely emptied by this operation. For example, if you tried to generate the slug of “-!=–#”, you would get an empty string.

There are other ways to do the same thing. Instead of replacing the leading and trailing dashes with nothing, you could create a regular expression which extracts everything between the leading and trailing slashes.

## Putting it all together

We could put our code into a function. This lets us pass a string to the function, and get a slug back out:

    function make_slug(str)
    {
        str = str.toLowerCase();
        str = str.replace(/[^a-z0-9]+/g, '-');
        str = str.replace(/^-+|-+$/g, '');
        return str;
    }

I hope that’s useful. Happy slugging!

<!--kg-card-end: markdown-->