---
layout: post
title: "Telemetry Server progress"
date: 2013-06-25 16:16
comments: true
categories: Mozilla
---

Just a quick post to update on progress with the Telemetry Server project.

First and foremost, the [telemetry-server code is now on github][3].

As of Friday, there is a basic prototype server up and running on an Amazon EC2 instance.

The prototype is able to accept submissions via HTTP, using URLs without or [with dimension components](https://bugzilla.mozilla.org/show_bug.cgi?id=860846). Submissions are then converted to the new [storage format][1] and saved to disk in the new [storage layout][2].

As part of the conversion process, the histograms in the payload are validated against the correct revision of `Histograms.json`. This file is automatically fetched from the Mozilla mercurial server, then cached locally. The `RevisionCache` class encapsulates this logic.

The prototype server in its current form is too monolithic, and needs to be split up such that receiving submissions via HTTP is independent from the remainder of the data pipeline.

The first part, recieving and logging HTTP submissions, is a well-understood problem and there are many existing ways so solve it.  Some options are:

*   Use the existing [Bagheera][4] server, and consume messages from Kafka.
*   Make more direct use of one of the many message queues (such as Amazon SQS, RabbitMQ, or Kafka)
*   Write new code to save raw payloads to a temporary storage area using a similar layout to the long-term [storage layout][2].

The second part is already in place and working, with code that should be quite easy to pull out and run separately. One additional piece of functionality that would be nice is to calculate realtime stats before/while data is being persisted.

Other than separating receiving and processing functionality, the next major task will be building a [MapReduce framework][5] to run on the Telemetry data. Initially, it will not be a full-blown cluster implementation, since reinventing that particular wheel is a huge task, but rather it will run on a single machine using multiple processes to parallelize map and reduce functionality.

One major advantage of the new [storage layout][2] is that MapReduce jobs will be able to filter the desired input data on a number of dimensions basically for free. Jobs that only need to look at a small subset of the data should be very efficient.

The first use case of the MapReduce framework is to produce the static data files for the new [telemetry frontend][6].

[1]: https://github.com/mreid-moz/telemetry-server/blob/master/StorageFormat.md "Storage Format"
[2]: https://github.com/mreid-moz/telemetry-server/blob/master/StorageLayout.md "On-disk Storage Layout"
[3]: https://github.com/mreid-moz/telemetry-server "Telemetry Server Repository"
[4]: https://github.com/mozilla-metrics/bagheera "Bagheera"
[5]: https://github.com/mreid-moz/telemetry-server/blob/master/MapReduce.md "Telemetry MapReduce"
[6]: https://github.com/tarasglek/telemetry-frontend "Telemetry Frontend"
