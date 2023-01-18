---
layout: post
title: Importing email folders into Gmail
date: '2018-10-07 10:30:04'
tags:
- email
- google
redirect_from:
- /importing-email-folders-into-gmail
---

Google has some useful documentation about [importing emails into Gmail](https://support.google.com/mail/answer/21289) via POP. However, the instructions don’t work very well if the messages in your old account are organised into folders. You will only end up importing messages which are directly in your inbox. I have found a few solutions though which I will outline in this post.

## Disclaimer

Please be aware that moving emails within or between accounts can be risky. It could result in information being lost in transit or misplaced. As such, if you follow this article, you do so entirely at your own risk. I am not an expert in email protocols or Google services, and I cannot offer technical support if something goes wrong. I am only sharing some tips which I have personally found useful.

## The problem

Gmail uses [POP](https://en.wikipedia.org/wiki/Post_Office_Protocol) to import the messages from your old account. Unfortunately, POP doesn’t support folders. As such, Gmail only sees the emails which are directly in your inbox.

Of course, Gmail doesn’t really support folders either. However, for this post, we will be treating Gmail labels as the closest equivalent.

## Solution 1: Inbox

This is the quickest, easiest, and generally most reliable solution. However, you lose all the folder information. You will need to manually reorganise all your messages in Gmail after importing them. If that’s not acceptable then consider solution 2 or 3 instead.

If you don’t mind losing the folder information, follow these steps:

1. In your old email account, move all messages directly into the inbox (they must not be in sub-folders).
2. Go through Gmail’s usual [POP-based import procedure](https://support.google.com/mail/answer/21289).
3. Wait for Gmail to finish downloading all messages. (This can take a long time. You can check progress from your Gmail settings screen.)
4. Check your spam folder to rescue any imported emails from there.

If you have a lot of emails then it’s usually best to do step 1 using a webmail interface if possible (i.e. access your email through a web-browser). It’s probably possible to do it using an email client such as Thunderbird too. However, it may be slower or less reliable.

## Solution 2: Inbox batches

This is very similar to solution 1, but it can be slightly easier to reorganise your messages in Gmail. This is probably the best option for importing folders containing more than a few hundred emails. One issue to be aware of though is that you may end up overlooking any messages which arrive during this process. Consider setting up a forwarding or filtering rule in your old account if possible to catch them.

1. In Gmail, create a custom label and apply it to everything currently in your inbox (this is so that you can find it later).
2. Archive everything in your inbox.
3. Enable [POP downloading](https://support.google.com/mail/answer/21289) of your old emails in Gmail (see the “Get all messages” -\> “Step 2” section at that link). Do not enable the archive option.
4. Wait for Gmail to finish downloading all messages. (This can take a long time. You can check progress from your Gmail settings screen.)
5. Label and archive all the newly imported messages in Gmail as appropriate.
6. Check your spam folder to rescue any imported emails from there.
7. In your old email account, move everything out of the inbox.
8. In your old email account, find a folder you want to import, and move the messages into your inbox.
9. Repeat steps 4 to 8 (inclusive) for every folder you want to import.
10. When you’ve finished importing everything, disable POP downloading in Gmail.
11. Find all the messages which were originally in your Gmail inbox (look for the custom label you created in step 1), and move them back into the inbox if desired.

As with solution 1, it’s usually best to use a webmail client to move emails around in your old account.

## Solution 3: IMAP client

This is the solution I used for most of my folders. It can be quite a laborious process. However, it’s probably the most straightforward option for folders which have no more than a few hundred messages each. You will need to use a computer for this, i.e. a PC or laptop running Windows, Linux, or Mac OS. If you only have access to a smartphone or a tablet, then you are probably stuck using solution 1 or 2.

1. Download and install an email client which supports IMAP, if you do not already have one. I recommend [Thunderbird](https://www.thunderbird.net), which is free.
2. Add your old email account to the email client (ensure you use IMAP not POP).
3. Add your Gmail account to the email client (ensure you use IMAP not POP). You may need to [enable IMAP in Gmail](https://support.google.com/mail/answer/7126229).
4. Click and drag a group of messages or an entire folder from your old account to your Gmail account.
5. Repeat step 4 until you’ve imported everything. (Wait for each operation to finish before trying to import more though.)

**CAUTION:** Gmail apparently limits you to uploading [300MB per hour or 500 MB per day](https://hiverhq.com/blog/gmail-and-google-apps-limits-every-admin-should-know/#Gmail) via IMAP. If you exceed the limits, you may be temporarily locked-out of your account.

That whole process probably looks a lot easier than it is. At best, it’s quite slow. I also found that the connection to Gmail timed-out regularly. That meant I had to try several times to finish uploading each folder. Moving the messages rather than copying them makes this easier as you can instantly see which ones haven’t been imported yet.

Another issue to be aware of is that Gmail will reject any message containing attachments above 25MB. It seems to stop the entire import when that happens. As such, don’t attempt to import large attachments at all. Most email clients will let you sort messages by size so you can easily identify the ones which will cause a problem.

## Conclusion

I’ve outlined a few ways of importing folder-based messages into Gmail. None of the solutions is ideal though. Part of the difficulty is that the email protocols were never designed for bulk operations. As such, fetching and moving large numbers of messages can be tricky and unreliable. I hope my suggestions above are helpful though. If you find a better solution then please let me know by leaving a comment below.

<!--kg-card-end: markdown-->