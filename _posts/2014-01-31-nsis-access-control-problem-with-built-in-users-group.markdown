---
layout: post
title: NSIS Access Control problem with built-in users group
date: '2014-01-31 21:19:41'
tags:
- windows
redirect_from:
- /nsis-access-control-problem-with-built-in-users-group
---

I was dealing with a subtle issue recently involving setting access permissions on Windows using [NSIS](http://nsis.sourceforge.net "Click to visit the NSIS webpage") (Nullsoft Scriptable Install System). It turned out that the problem was not with NSIS at all. Rather, it was a misunderstanding on my part regarding an unexpected quirk in Windows. Hopefully this post will help anybody who encounters a similar issue.

The installer was supposed to enable read/write permissions for all users on certain important files and registry keys. However, some users were finding these files/keys were not accessible and it was preventing the software from working correctly.

## Language barrier

I eventually realised that the affected users were all running Windows 7 systems which had not been setup in English. The file permissions were being set like this:

```
AccessControl::GrantOnFile "$INSTDIRfile.foo" "BUILTINUSERS" "GenericRead + GenericWrite"
```

Older versions of our installer had previously specified user group `(BU)` instead of `BUILTINUSERS`. However, the `(BU)` only works on Windows XP. As such, it had been changed to `BUILTINUSERS`, which seemed to work on Windows XP, Vista, 7, and 8.

Unfortunately, it turns out that `BUILTINUSERS` is a localised string, meaning it’s in whatever language was used when Windows was originally installed on the system. That means that specifying it in English in the installer script will fail on non-English systems.

In retrospect, that’s probably very obvious to anybody who administers Microsoft systems on a daily basis, but it took me by surprise!

## Security Identifiers

The correct approach is to use the newer security identifiers (SIDs). These are not localised; i.e. they are the same no matter what the original system language is. According to [Microsoft’s documentation](https://learn.microsoft.com/en-GB/windows-server/identity/ad-ds/manage/understand-security-identifiers), the equivalent SID for the built-in users group is: `(S-1-5-32-545)`

The reason I didn’t originally use that was because it’s not supported by Windows XP. That version of the OS is still being used by several of my employer's long-standing customers. The assumption was that trying to set a file permission using an unsupported security identifier would prevent the installation from working.

That assumption appears to be wrong. Based on several tests, the individual file permission command silently fails and the installer moves on without any problem.

## The Solution

The simplest way to support Windows XP and Vista onwards in a single installer is to use both types of language-independent user group identifier. For example, instead of the “BUILTINUSERS” line shown earlier, the installer now has these two lines:

```
AccessControl::GrantOnFile "$INSTDIRfile.foo" "(BU)" "GenericRead + GenericWrite"
AccessControl::GrantOnFile "$INSTDIRfile.foo" "(S-1-5-32-545)" "GenericRead + GenericWrite"
```

So far, that appears to work correctly, supporting users on Windows XP and on Windows Vista onwards. I haven’t observed any side effects from setting the permissions twice, or from either one failing on other OS versions.
