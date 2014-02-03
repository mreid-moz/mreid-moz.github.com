---
layout: post
title: "Deploying the next Telemetry Server"
date: 2013-10-02 14:15
comments: true
categories: Mozilla
---

Yesterday was our target for cutting over to the "rebooted" telemetry server.
Despite some last-minute travel (I spent Monday en route from Nova Scotia to
San Francisco), I'm happy to report that the switch went rather smoothly on
Tuesday! More details on the required changes can be found in [Bug 921161][1].

Since my last update, there have been a few last-minute code changes to get
things in ship-shape for deployment. The bulk of those changes were to the
scripts used to provision machines on Amazon's EC2 infrastructure, but there
was one more structural change of note.

The logic for processing incoming submissions (that's the "validation,
conversion, and compression" part) was previously controlled by a master
process which would launch a worker node to do the actual processing. Without
an easy way for masters to co-ordinate, it was difficult to launch extra
workers in cases where the rate of processing was not keeping up, since each
master expected its worker to process all available data.

The solution was to switch to using a [queue][2] to keep track of data
available to be processed, and having the worker nodes claim data from the
queue. This results in a nicely decoupled architecture, where starting up more
workers (or killing off idle ones) is clean and easy.

Anyways, getting back to the main point... The cutover is complete, and the
Telemetry submissions are now going to "The Cloud!"

It turns out that the node.js version of the web server is efficient enough to
allow us to handle the entire volume of traffic using only a pair of "t1.micro"
nodes in EC2 (behind a load balancer).  Pretty slick.

So far, running on AWS has been pretty nice. The Elastic Load Balancers make it
nearly-trivial to add or remove nodes from the pool, and include useful (if
limited) monitoring. On the HTTP-serving nodes themselves, we have some more
detailed and application-specific monitoring using [Heka][3]. The [boto][4]
library makes it very easy to provision new nodes using python.

Now that the Telemetry Server is out in the wild, the next step is to get the
[new Dashboard][5] playing nicely with the new data source. Jonas Finnemann
Jensen is working on that.

There's still more work to do once the dashboard integration is complete,
including finishing up the C++ port of the "process incoming" code (which will
hopefully provide a large speedup compared to the current python
implementation), migrating the provisioning over to Amazon Cloud Formation,
creating a frontend for managing/running Telemetry MapReduce jobs, and
[exporting the historic data][6] from the previous Hadoop backend into the new
S3 storage.

[1]: https://bugzilla.mozilla.org/show_bug.cgi?id=921161
[2]: http://aws.amazon.com/sqs/
[3]: http://hekad.readthedocs.org/en/latest/
[4]: http://boto.s3.amazonaws.com/index.html
[5]: http://telemetry-dash.mozilla.org
[6]: https://bugzilla.mozilla.org/show_bug.cgi?id=922745

