---
layout: post
title: "Current state of Telemetry Analysis"
date: 2013-11-06 10:45
comments: true
categories: Mozilla
---

The Telemetry Server has been deployed on AWS for just over a month now, so it's
time for an update.

The server code repository has been moved into the [_Mozilla_ github group][1],
and the `mreid-moz` repo forwards there, so the change should be seamless.

The Telemetry dashboards have also moved! They are now located at
[telemetry.mozilla.org][2], nice and easy to remember.

Moving on to more interesting news, anyone with an `@mozilla.com` email can now
run their own Telemetry analysis jobs in the cloud. The procedure is still very
much in alpha/beta state, but if you've got a question that can be answered
using Telemetry data, you're in luck.

Jonas has built [a mechanism for provisioning a ubuntu server as an Amazon EC2
instance][4]. These machines (`m2.4xlarge` in EC2 terms) have read-only
permission and a fast connection to Telemetry data stored in S3. Each machine
will be available for 24 hours, and will cost about $40USD to run. If you don't
need it for the full day, you can kill it early by following the instructions in
the webapp below.

Here's how it works:

-  Visit the analysis provisioning dashboard at [telemetry-dash.mozilla.org][3]
   and sign in using Persona (with an `@mozilla.com` email address as mentioned
   above). 
-  Enter some details. The `Server Name` field should be a short descriptive
   name, something like 'mreid chromehangs analysis' is good. Upload your
   SSH public key (this allows you to log in to the server once it's started up)
-  Click `Submit`.
-  A Ubuntu machine will be started up on Amazon's EC2 infrastructure. Once it
   has started up, you can SSH in and run your analysis job. Reload the webpage
   after a couple of minutes to get the full SSH command you'll use to log in.
-  **Make sure to download your results** when you're finished! Your analysis
   machine will automatically get killed after 24 hours.

Ok, that's all well and good, but what _is_ an analysis job?

The easiest (and probably most familiar to anyone who has worked with Telemetry
data in the past) is to run a MapReduce job.

This requires a bit of setup on the machine you provisioned above:

-  Clone the `telemetry-server` github repository:
```
    $ sudo apt-get install git
    $ git clone https://github.com/mozilla/telemetry-server.git
```
-  Set up some working directories:
```
    $ sudo mkdir /mnt/telemetry
    $ sudo chown ubuntu:ubuntu /mnt/telemetry
    $ mkdir /mnt/telemetry/work
```
-  Create an input filter to match the data you want. Look at the examples at
   `mapreduce/examples/*.json`. Here is [a reasonably selective one][5].
-  Create a new analysis script, or run one of the example ones:
```
    $ cd ~/telemetry-server
    $ python -m mapreduce.job mapreduce/examples/osdistribution.py \
       --input-filter /path/to/filter.json \
       --num-mappers 16 \
       --num-reducers 4 \
       --data-dir /mnt/telemetry/work \
       --work-dir /mnt/telemetry/work \
       --output /mnt/telemetry/my_mapreduce_results.out \
       --bucket "telemetry-published-v1"
```

A few notes for successful jobs.

- Try to keep the amount of data you're crunching to a minimum while you're
  developing your job. It'll save time, and prevent the system from running out of
  memory. A good start is to process just a single day's `nightly` data.
- If you do run out of memory, try increasing the number of mappers and reducers.
- After the first run, you can point the "--data-dir" argument at
  `<work-dir>/cache` and add the "--local-only" parameter to skip downloading
  files from S3 every time
- Give us a heads-up in #telemetry and we'll tell you about any other caveats.

One final note - the Telemetry MapReduce framework is a simple way to download
the set of records you are interested in, and _do something_ for each record in
that data set.

If you don't want to do your analysis with this framework, you can just use it
to download the data (or even skip it altogether and download data using the AWS
command-line tools directly). Once the data has been downloaded to the machine,
you're free to analyze it using whatever other language / tools you're
comfortable with.

Happy Data Crunching!

[1]: https://github.com/mozilla/telemetry-server
[2]: http://telemetry.mozilla.org
[3]: http://telemetry-dash.mozilla.org
[4]: https://github.com/mozilla/telemetry-server/blob/master/analysis/debug-service/server.py
[5]: https://github.com/mozilla/telemetry-server/blob/master/mapreduce/examples/filter_saved_session_Fx_prerelease.json
