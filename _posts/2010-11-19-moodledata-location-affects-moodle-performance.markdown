---
layout: post
title: moodledata location affects Moodle performance
date: '2010-11-19 11:37:48'
---

I’ve been working with [Moodle](https://moodle.org) for several years now, primarily as part of the [SLOODLE](http://www.sloodle.org) project. However, one thing that has always been a source of frustration is how slow Moodle often seems on my local Windows Vista PC in the office. (Sadly, I don’t have the option of upgrading to Win7.)

As it turns out, the performance problem was entirely caused by where I had placed my “moodledata” folder, and there is an easy solution.

## Standard advice

The advice that is always given is to keep the “moodledata” folder away from your webserver folder. This is to ensure that nobody can access the data directly. It should only be accessible via Moodle. On my public server, I pay close attention to this because it’s an important security point (albeit not possible on all webservers).

However, for my test installations on my local machine, I have a tendency to keep the data folder alongside the main Moodle folder because that makes it a little easier to use. Nobody can access it from outside my PC, and there’s no sensitive data in it, so securing the installation isn’t important.

As it turns out, that’s actually the cause of my performance problem. It seems that Moodle automatically checks to see if the data folder is web-accessible. This is a great feature for ensuring admins have secure installations, but something about this check seems to go very slow on Windows Vista, and it sometimes seems to break installation/upgrade of Moodle and its modules.

I’ve never had the same problem on Win XP or 7, despite using the same webserver configuration. As such, I suspect it’s somehow related to the Vista file permissions system or User Access Control, which have caused a number of problems for other software. Even setting the moodledata folder to be fully accessible by all users didn’t solve the problem.

## The solution

At the risk of being obvious, the solution is to follow the standard advice. That is, make sure your moodledata folder is not directly web-accessible. For example, if you keep your Moodle folder here:

- `c:\wamp\www\moodledata`

Then you should keep your moodledata folder here:

- `c:\wamp\moodledata`

The exact paths will vary from one webserver configuration to the next. Personally, I like to use [WampServer](http://www.wampserver.com/en) when I’m working on a Windows system. Regardless, the principle is this: if “www” is your webserver folder, then put moodledata in the folder above it. That should resolve the performance issue, seemingly because Moodle will no longer be able to test whether or not it is accessible from the web.
