---
layout: post
title: 'PHP gotcha: Content-Type matters!'
date: '2010-04-17 19:09:32'
tags:
- php
redirect_from:
- /php-gotcha-content-type-matters
- /php-gotcha-content-type-matters/
---

I encountered a frustrating problem regarding HTTP content type headers today. It massively broke the design of my site in Firefox, but didn’t seem to affect anything in Internet Explorer. The lesson learned is to be careful where you set the Content-Type header in PHP.

## Background

I’ve been working on a new website which divides the CSS into two separate files: one which governs layout, and one which governs aesthetic styles like colours and fonts. In both IE and FF, the layout itself was fine. However, in FF the aesthetic stylesheet wasn’t having any effect at all. It’s as though it wasn’t being included. I checked all the usual CSS/HTML problems I could think of, but I couldn’t find anything.

It’s worth noting that the aesthetic stylesheet was actually a PHP script. This was because I wanted to be able to swap-out colour schemes dynamically during development for testing different ideas. I had been careful to tell PHP to push a suitable Content-Type header at the top of the file, before anything else was sent:

```php
header("Content-Type: text/css; charset=utf-8");
```

Strangely, the CSS data nonetheless seemed to be detected as “text/html” by the browser.

## The problem

Immediately _after_ the header command shown above, I was including our site’s general “config” script. This is what allowed us to grab current colour scheme details. Unfortunately, I had unthinkingly put a generic Content-Type header command in there for convenience. It’s included on all our regular HTML pages, so naturally it was set to “text/html”.

Firefox saw the second Content-Type header as being the correct one, and therefore ignored the CSS script. Evidently, IE doesn’t really care about accurate content types, which is why it seemed to work there.

## The solution

Set the Content-Type header correctly, and only set it once.
