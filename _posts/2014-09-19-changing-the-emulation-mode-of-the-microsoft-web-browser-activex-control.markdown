---
layout: post
title: Changing the emulation mode of the Microsoft Web Browser ActiveX control
date: '2014-09-19 18:12:27'
tags:
- windows
redirect_from:
- /changing-the-emulation-mode-of-the-microsoft-web-browser-activex-control
- /changing-the-emulation-mode-of-the-microsoft-web-browser-activex-control/
---

I’ve been working on an MFC project which embeds a basic web-browser component in a dialog. It does this using a Microsoft Web Browser ActiveX component. The control should hook into whatever version of Internet Explorer (IE) is running on the system. However, it always seemed to fall-back on IE7 emulation mode for me, meaning a lot of modern standards-compliant HTML wasn't being displayed properly.

I was able to fix this by setting a registry key.

## Registry value

You can modify this emulation behaviour by adding a registry value, telling the component which version of IE to emulate for a given executable. It can either go under `HKEY_LOCAL_MACHINE` or `HKEY_CURRENT_USER`, at the following location for a 32-bit system:

`Software\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_BROWSER_EMULATION`

Or this location on a 64-bit system:

`Software\Wow6432Node\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_BROWSER_EMULATION`

Add a new `DWORD` with the same name as your executable (e.g. “myprogram.exe”). The value of the `DWORD` identifies which browser version to use:

| Desired IE version | DWORD value (decimal) | DWORD value (hex) |
| ------------------ | --------------------- | ----------------- |
| IE7                | 7000                  | 0x1B58            |
| IE8                | 8000                  | 0x1F40            |
| IE9                | 9000                  | 0x2328            |
| IE10               | 10000                 | 0x2710            |
| IE11               | 11000                 | 0x2AF8            |

The trend is pretty obvious here. Presumably IE12 (if there is one) will be 12000 (decimal), and so on. I’d advise against putting in a value higher than you can currently test though as it could have unexpected side effects in future.

It’s important to note that this only affects the default document mode. If a page contains non-standard quirks then the browser’s compatibility mode may come into play. That could cause the emulation to drop back to a previous version. The best way to prevent this is to use up-to-date standards-compliant HTML and CSS.

## Limitations

Since the registry key is specified only by the program name (not the path), there is obviously scope for conflicts here. For example, if two different pieces of software both call their main executable “app.exe” then there is no way to distinguish between them in the registry setting. This could be a particular problem if you expect your users to have multiple versions of your software at the same time.

I haven’t found a way round this, so my only advice is to make sure you use unique executable names.

## Support on older systems

Older systems such as Windows XP can’t run the more recent versions of Internet Explorer. However, the registry codes above will still work. As far as I can tell, the ActiveX control just uses the latest version of IE it can find, up to and including whatever you specify in the registry setting.

For example, if a user only has IE8 on their system, it won’t cause a problem if you specify the code for IE11 in the registry. The ActiveX control will happily use IE8 instead.
