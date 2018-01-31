---
layout: post
comments: true
title: Adding a column is unsafe
date: 2018-01-31 8:20:30
categories: postgres
short_description: Beware of adding a column in postgres
image_preview: /images/macdad.gif
---

I've been burned by migrations in production many times now.

Because of this, my current team goes to great lengths to make sure migrations don't go unnoticed.
We make sure code is always compatible without the migration being run in case we have to rollback.
This takes time and more deploys, but up until now it's worked very well.
We use the [strong_migrations](https://github.com/ankane/strong_migrations) gem, which forces us to remember what's
"safe" and "unsafe".

If you google unsafe migrations, a lot of the resources state that adding a new column is fine.
Including, the strong_migrations gem. This is not, strictly speaking, true.

We ran a migration in the morning to add a new column to a critical table, when usage is the highest.
This is also when we have several long running reporting jobs in mid-execution.
Running this migration caused the application to lock up. So what happened?

This [article](http://www.databasesoup.com/2013/11/alter-table-and-downtime-part-ii.html) by postgres contributor Josh Berkus details exactly what happened to us.

After investigating, we noticed a single query that could take up to 40 minutes to run.
When you add a new column, postgres issues an `ACCESS EXCLUSIVE` lock for that table (more [here](https://www.postgresql.org/docs/9.5/static/sql-altertable.html)).
That locks *reads* and *writes*.

Once this lock has been requested, *all* queries will queue up behind it.
If a 40 minute query is running, everything will back up behind the `ALTER` statement and you'll have 40 minutes of downtime. No bueno. Depending on your read/write volume, your database will likely cripple under this weight, like ours did. (We have ~1000 reads/second and ~250 writes/second).

So, what can we do? Fixing these long running queries is the answer. Usually, adding a column is safe because it runs
in miliseconds and if you have all short running queries, the queue isn't going to build up behind the lock.

I think the biggest problem right now is that so many resources on the internet state that adding a column is a worry free operation.
While 40 minutes is extreme, I think *most* large applications that have been in development a while will have
long running queries and it's a real risk to add a column in production.

You've been warned.
