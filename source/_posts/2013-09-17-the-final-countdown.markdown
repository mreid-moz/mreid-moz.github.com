---
layout: post
title: "The Final Countdown"
date: 2013-09-17 14:05
comments: true
categories: Mozilla
---

Release day is rapidly approaching. We're targeting October 1st for the release
of the first version of the telemetry reboot. Only two weeks left - exciting
times!

There's now a tracking bug that should list everything that needs to be done
before things go live: [Bug 911300][1].

The code has changed a fair bit since my last update, the most noticeable
modification being that the HTTP server is now using node.js instead of a
python+flask server.

Here is the basic architecture diagram for the system:
![Telemetry Data Flow diagram][2]

Data flows into an `incoming` bucket, which can then be processed separately by
one or more processing nodes, each of which publishes finalized data to a
`published` bucket.

The MapReduce code then reads from the `published` location for data analysis.

Other changes include updating the ["process incoming data"][3] code to take
advantage of multiple processors, though it appears that python performance is
still a major bottleneck. Fortunately Mike Trinkala is working on porting the
conversion code to C++.

Keep an eye on the bug above for up-to-the-minute progress information, or as
always, feel free to drop by `#telemetry` on `irc.mozilla.org`.

[1]: https://bugzilla.mozilla.org/show_bug.cgi?id=911300
[2]: https://raw.github.com/mreid-moz/telemetry-server/master/docs/data_flow.png
[3]: https://github.com/mreid-moz/telemetry-server/blob/master/process_incoming_mp.py
