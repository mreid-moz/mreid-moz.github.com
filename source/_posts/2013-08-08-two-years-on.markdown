---
layout: post
title: "Two Years In"
date: 2013-08-08 11:47
comments: true
categories: Mozilla
---

I figured since I missed it last year, I should post something on my
Mozillaversary!

I started on the Metrics team on August 8th, 2011, and more recently moved over
to the Performance team. Lots of fun stuff to learn and do, and I'm happier
than ever that I joined Mozilla.

Two orders of business in this post. First, another status update on the 
Telemetry Reboot project.

Things are basically feature-complete on the server now! Very exciting. The
high-altitude view looks like this:

Data comes in as HTTPS submissions, and is saved to disk in its raw form. These
files are converted to the new Histogram storage format, compressed, then sent
to Amazon S3 for permanent storage.

You can run MapReduce jobs on the S3 data (though at the moment there's a lot
more time spent on data transfer than I would like). If you run a MapReduce job
on the server node, you can also include up-to-the-minute data that has not yet
been exported to S3.

Now that the prototype (which reminds me a bit of this description of the
[Hideous Creature][1] **Warning:** strong language) is working, the next step is to
benchmark things to make sure we can handle release-level volumes.

The second order of business is an interesting thing I learned about Python
File I/O that I thought I'd share.

Normal file writes are not atomic, so if you have a situation where multiple
concurrent processes are appending lines to the same file, it's possible (and
in fact very likely) to end up with garbled lines. The Telemetry storage format
requires that each record be on its own line, so this caused real problems
during testing.

The solution: use `io.open(...)`.

For example, doing this will produce non-atomic writes:
```
fout = open(filename, "a")
fout.write(some_content)
fout.close()
```

So `some_content` could actually be written out in multiple disk operations,
with other writes by other processes in between.

Instead, to achieve atomic writes, you would use something like this:
```
with io.open(filename, "a") as fout:
    fout.write(some_content)
```

Maybe this is common knowledge for Python folks, but I hope it helps.

[1]: http://www.youtube.com/watch?feature=player_detailpage&v=bzkRVzciAZg&t=158
