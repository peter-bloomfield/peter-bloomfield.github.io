---
layout: post
title: Emit audible warning from a batch script
date: '2015-10-22 11:59:50'
tags:
- batch
- windows
---

Windows Batch (\*.bat) scripting is archaic and painful, but occasionally useful for quick bits of environment setup. It’s not always obvious to the user when something has gone wrong though as it’s easy to lose the information amidst other text which scrolls by. An audible warning can be a useful alternative to draw attention to a problem.

On many systems, this can be done quite simply by outputting the ASCII bell character, which has character code 7. Here’s what it might look like in a batch script (depending on your editor):

    ECHO •

It should hopefully be possible to copy-and-paste that directly into a batch script using your favourite plain text editor.

To enter it manually, first type in the “ECHO” followed by a space. Next, ensure your Num Lock is enabled, hold the left **Alt** key on your keyboard, and press the 7 key on your number pad. Save it to file (e.g. as “test.bat”), then double-click that file to test it.

Please note that your results may vary here. Inputting ASCII characters this way may not work on all editors. It seems to work for me with [Notepad++](https://notepad-plus-plus.org/) though (an editor I highly recommend), and even with Windows’ regular Notepad. You may also find that the beep won’t sound on some systems, depending on how they are configured.

<!--kg-card-end: markdown-->