---
layout: post
title: 'ADOdb function: GetUpdateSQL'
date: '2010-01-21 02:14:58'
tags:
- php
- databases
redirect_from:
- /adodb-function-getupdatesql
- /adodb-function-getupdatesql/
---

I have been getting myself familiar with [ADOdb](http://adodb.org) by using it in a personal web-development project. During my experiments, I ran across a member function called `GetUpdateSQL()`. In principle, it looks like a very useful function. Given a recordset which you’ve previously retrieved in a query, and given some desired changes, it will generate an UPDATE query string for you. However, I have found it to be a little disappointing for my needs.

## What changes? Where?

A slightly counter-intuitive aspect of the function is the way you give it changes. It takes two parameters. The first is the recordset you want to make changes to, but crucially this has to be the _original_ recordset without any modifications. You specify your modifications using an associative array (associating the name of the column with the desired new value). The query and update might look like this:

```php
$recordset = $db->Execute("SELECT * FROM users WHERE id = 5");
$mychanges = array('name' => 'Fred Flinstone');
$sql = $db->GetUpdateSQL($recordset, $mychanges);
$db->Execute($sql);
```

Passing the changes in as a separate array is a slight annoyance, although I can see why it is done. The original recordset is used to determine whether any data changes are actually being requested, and so no unnecessary changes are sent to the database. Efficiency is good.

However, it might be nice if something was built-in to the recordset class which would track the changes, if desired. The modified recordset object could then be passed straight back to the database connection object to say “write these changes to the database”.

Of course, in that very simple example it would be overkill to use the `GetUpdateSQL()` function. It’s for illustration purposes only though.

## Which records are modified?

For me, this is the really problematic aspect of this function. If your recordset contains multiple records (or rows) then the update is always applied to _all_ of them. As far as I can tell, you cannot even extract a single record from the recordset and only update that one. If you want to update each record separately, then you first have to execute multiple queries to obtain a recordset for each one. That is obviously very inefficient, so you just have to construct the UPDATE queries yourself.

It has to be noted that it is entirely common to be unable to update multiple records in a single query. Typically you can just specify a single WHERE clause. As such, the underlying queries would still have to resolve to multiple updates anyway. However, it would be nice if the wrapper could provide some extra functionality here.

## What would I prefer?

There are a couple of possibilities. I have already mentioned my ideal solution, whereby the recordset just gets modified and updated. Alternatively, one very simple solution would be the ability to generate the update SQL for only the ‘current’ record in the recordset. For example, if you are iterating through all records in a loop, you could auto-generate SQL to update just the current one.

Perhaps a slightly better alternative would be the ability to specify multiple arrays of changes. You wouldn’t just supply an associative array of “column =\> value” changes to be applied to all records. Rather, you would supply an array of those associative arrays; i.e. one for each record. Being able to group all the updates together into a batch like this would provide some optimisation possibilities.

Anyway, those are just my ideas. I appreciate that ADOdb is primarily concerned with portability of database code, and it does that very well. However, it would be nice to see some more time-saving features like these.
