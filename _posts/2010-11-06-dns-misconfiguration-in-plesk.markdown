---
layout: post
title: DNS misconfiguration in Plesk
date: '2010-11-06 02:18:32'
redirect_from:
- /dns-misconfiguration-in-plesk
- /dns-misconfiguration-in-plesk/
---

I run a VPS (Virtual Private Server) through the 1&1 webhost, and was trying to create a new subdomain today using the Plesk 9.5.2 admin interface. This is an easy task which I have done many times before, but this time it failed, and gave me the following error message:

```
Service is not available now, probably your Plesk is misconfigured.
Contact Your provider for details.
Internal Plesk error occurred: dnsmng::update() failed: dnsmng failed: dnsmng: Empty fields 'host' or 'opt' in PTR record
```

Panic! I had no idea what to do, and web-searches provided very little information. However, I was able to find enough information to figure out the solution.

I don’t know exactly what caused the problem (most likely a glitch last time I updated the Plesk software), but the problem was very simply a couple of blank fields in the database. It turned out to be an easy fix.

## Step 1: accessing the Plesk database

1. Login to Plesk as admin
2. Access the “Database Servers” section (there’s a link to this on the “Home” page of my control panel, but this may not be the case on all setups)
3. Click “Local MySQL server”
4. Click the “Databases” tab
5. Click the “Webadmin” button at the top
6. Click the database called “psa”
7. Click the table called “dns\_recs” (on the left-hand column)
8. Click the “Browse” tab

This solution worked for me as admin, but it may not work for every installation. Unfortunately, the root MySQL password is not a piece of information I have for my installation (it’s buried in Plesk somewhere), but if you know yours then that’s a viable alternative if you can get phpMyAdmin or similar working.

## Step 2: modifying the data

In the “dns\_recs” table, you’ll hopefully see a number of DNS entries. You should see one or more with a type called “PTR”, and you’re looking for any such record where there are blank spaces for the “displayHost” and “host” entries. In my case, it was just the very first record that was a problem. I was able to copy the “displayHost” and “host” values from a different PTR row, and insert them into the problem row.

In my case, the missing value was just the IP address of my VPS (I only have one IP address). In your case, it may be different, especially if you have multiple IP addresses. I recommend trying your main IP address (if you have one).

## Done!

Once again, I’m afraid I don’t know exactly what caused the problem. Regardless, the solution seems to be fixing the empty data in the `dns_recs` table. If you find a different (perhaps better) solution to the same problem, then please share it in the comments below.
