---
layout: post
title: "Scheduling Telemetry Analysis"
date: 2014-03-14 14:57
comments: true
categories: Mozilla
---

In a [previous post][1], I described how to run an ad-hoc analysis on the
Telemetry data. There is often a need to re-run an analysis on an ongoing
basis, whether it be for [powering dashboards][2] or just to see how the
[data changes over time][3].

We've rolled out a new version of the
[Self-Serve Telemetry Data Analysis Dashboard][4] with an improved interface
and some new features.

Now you can schedule analysis jobs to run on a daily, weekly, or monthly basis
and publish results to a public-facing Amazon S3 bucket.

Typical Scheduling
---

Here's how I expect that the analysis-scheduling capability will normally be
used:

1.  Log in to [telemetry-dash.mozilla.org][4]
1.  Launch an analysis worker in the cloud
1.  Develop and test your analysis code
1.  Create a wrapper script to:
    -   Do any required setup (install python packages, etc)
    -   Run the analysis
    -   Put output files in a given directory relative to the script itself
1.  Download and save your code from the worker instance
1.  Create a tarball containing all the required code
1.  Test your wrapper script:
    -   Launch a fresh analysis worker
    -   Run your wrapper script
    -   Verify output looks good
1.  Head back to [the analysis dashboard][4], and schedule your analysis to run
    as needed

Dissecting the "Schedule a Job" form
---
-   **Job Name** is a unique identifier for your job. It should be a short,
    descriptive string. Think "what would I call this job in a code repo or
    hostname?" For example, the job that runs the data export for the
    [SlowSQL dashboard][5] is called `slowsql`. You can add your username to
    create a unique job name if you like (ie. `mreid-slowsql`).
-   **Code Tarball** is the archive containing all the files needed to run your
    analysis. The machine on which the job runs is a bare-bones Ubuntu machine
    with a few common dependencies installed (git, xz-utils, etc), and it is up
    to your code to install anything extra that might be needed.
-   **Execution Commandline** is what will be invoked on the launched server. It
    is the entry point to your job. You can specify an arbitrary Bash command.
-   **Output Directory** is the location of results to be published to S3.
    Again, these results will be publicly visible.
-   **Schedule Frequency** is how often the job will run.
-   **Day of Week** for jobs running with `daily` frequency.
-   **Day of Month** for jobs running with `monthly` frequency.
-   **Time of Day (UTC)** is when the job will run. Due to the distributed
    nature of the Telemetry [data processing pipeline][6], there may be some
    delay before the previous day's data is fully available. So if your job is
    processing data from "yesterday", I recommend using the default vaulue of
    Noon UTC.
-   **Job Timeout** is the failsafe for jobs that don't terminate on their own.
    If the job does not complete within the specified number of minutes, it will
    be forcibly terminated. This avoids having stalled jobs run forever (racking
    up our AWS bill the whole time).


Example: SlowSQL
---
A concrete example of an analysis job that runs using the same framework is the
[SlowSQL export][7]. The `package.sh` script creates the code tarball for this
job, and the `run.sh` script actually runs the analysis on a daily basis.

In order to schedule the SlowSQL job using the above form, first I would run
`package.sh` to create the code tarball, then I would fill the form as follows:

-   **Job Name**: `slowsql`
-   **Code Tarball**: select `slowsql-0.3.tar.gz`
-   **Execution Commandline**: `./run.sh`
-   **Output Directory**: `output` - this directory is created in `run.sh` and
    data is moved here after the job finishes
-   **Schedule Frequency**: `Daily`
-   **Day of Week**: leave as default
-   **Day of Month**: leave as default
-   **Time of Day (UTC)**: leave as default
-   **Job Timeout**: `175` - typical runs during development took around 2 hours,
    so we wait just under 3 hours

The [daily data files][8] are then published in S3 and can be used from the
Telemetry SlowSQL Dashboard.

Beyond Typical Scheduling
---
The job runner doesn't care if your code uses the [python MapReduce][6]
framework or your own hand-tuned assembly code. It is just a generalized way to
launch a machine to do some processing on a scheduled basis.

So you're free to implement your analysis using whatever tools best suit the
job at hand.

The sky is the limit, as long as the code can be packaged up as a tarball and
executed on a Ubuntu box.

The pseudo-code for the general job logic is
```bash
fetch $code_tarball
tar xzvf $code_tarball
`$execution_commandline`
cd $output_directory
publish --recursive ./ s3://public/$job_name/data/
```

Published Output
---

One final reminder: Keep in mind that the results of the analysis will be made
**publicly available** on Amazon S3, so be absolutely sure that the output from
your job does not include any sensitive data.

Aggregates of the raw data are fine, but it is very important not to output the
raw data.





[1]: http://mreid-moz.github.io/blog/2013/11/06/current-state-of-telemetry-analysis/
[2]: http://telemetry.mozilla.org
[3]: https://bugzilla.mozilla.org/show_bug.cgi?id=965707
[4]: http://telemetry-dash.mozilla.org/
[5]: http://telemetry.mozilla.org/slowsql
[6]: https://raw.github.com/mreid-moz/telemetry-server/master/docs/data_flow.png
[7]: https://github.com/mozilla/telemetry-server/tree/master/mapreduce/slowsql
[8]: https://s3-us-west-2.amazonaws.com/telemetry-public-analysis/slowsql/data/slowsql20140308.csv.gz
