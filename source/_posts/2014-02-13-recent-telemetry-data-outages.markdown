---
layout: post
title: "Recent Telemetry Data Outages"
date: 2014-02-13 16:55
comments: true
categories: Mozilla
---

There have been a few data outages in Telemetry recently, so I figured some followup and explanation are in order.

Here's a graph of the submission rate from the `nightly` channel (where most of the action has taken place) over the past 60 days:

![Heka Graph of nightly submissions for the past 60 days][1]

The x-axis is time, and the y-axis is the number of submissions per hour.

Points **A** (December 17) and **B** (January 8) are false alarms, and were just cases where the stats logging itself got interrupted, and so don't represent data outages.

Point **C** (January 16) is where it starts to get interesting. In that case, Firefox nightly builds stopped submitting Telemetry data due to a change in event handling when the Telemetry client-side code was [moved from a .js file to a .jsm module][3].  The resolution is described in [Bug 962153][2].  This outage resulted in missing data for nightly builds from January 16th through to January 22nd.

As shown on the above graph, the submission rate dropped noticeably, but not anywhere close to zero.  This is because not everyone on the nightly channel updates to the latest nightly as soon as it's available, so an interesting side-effect of this bug is that we can see a glimpse of the rate at which nightly users update to new builds.  In short, it looks like a large portion of users update right away, with a long tail of users waiting several days to update. The effect is apparent again as the submission rate recovers starting on January 22nd.

The second problem with submissions came at point **D** (February 1) as a result of changing the client Telemetry code to [use OS.File for saving submissions][5] to disk on shutdown. This resulted in a more gradual decrease in submissions, since the ["saved-session" submissions were missing][4], but "idle-daily" submissions were still being sent. This outage resulted in partial data loss for nightly builds from February 1st through to February 7th.

Both of these problems have been limited to the `nightly` channel, so the actual volume of submissions that were lost is relatively low. In fact, if you compare the above graph to the graph for all channels:

![Heka Graph of all submissions for the past 60 days][7]

The anomalies on January 16th and February 1st are not even noticeable within the usual weekly pattern of overall Telemetry submissions (we normally observe a sort of "double peak" each day, with weekends showing about 15-20% fewer submissions). This makes sense given that the `release` channel comprises the vast majority of submissions.

The above graphs are all screenshots of our [Heka][9] aggregator instance. You can look at a [live view of the submission stats][8] and explore for yourself. Follow the links at the top of the iframes to dig further into what's available.

There was a third data outage recently, but it cropped up in the server's data validation / conversion processing so it doesn't appear as an anomaly on the submission graphs. On February 4th, the `revision` field for many submissions started being rejected as invalid. The server code was expecting a `revision` URL of the form `http://hg.mozilla.org/...`, but was getting URLs starting with `https://...` and thus rejecting them. Since `revision` is required in order to determine the correct version of `Histograms.json` to use for validating the rest of the payload, these submissions were simply discarded. The change from `http` to `https` came from [a mostly-unrelated change to the Firefox build infrastructure][10]. This outage affected both nightly and aurora, and caused the loss of submissions from builds between February 4th and when the [server-side fix][14] landed on Februrary 12th.

------------


So with all these outages happening so close together, what are we doing to fix it?

Going forward, we would like to be able to completely avoid problems like the overly-eager rejection of payloads during validation, and in cases where we can't avoid problems, we want to detect and fix them as early as possible.

In the specific case of the `revision` field, we are adding [a test that will fail if the URL prefix changes][11].

In the case where the submission rate drops, we are adding [automatic email notifications][6] which will allow us to act quickly to fix the problem. The basic functionality is already in place thanks to Mike Trinkala, though the anomaly detection algorithm needs some tweaking to trigger on things like a gradual decline in submissions over time.

Similarly, if the ["invalid submission" rate][12] goes up in the server's validation process, we want to add [automatic alerts][13] there as well.

With these improvements in place, we should see far fewer data outages in Q2 and beyond.


### Last minute update
While poking around at various graphs to document this post, I noticed that [Bug 967203][4] was still affecting `aurora`, so the fix has been uplifted.


[1]: images/telemetry_nightly20140213.png "Nightly submission rate for the past 60 days"
[2]: https://bugzilla.mozilla.org/show_bug.cgi?id=962153 "Bug 962153 - Drop in telemetry submissions since 2014-01-15"
[3]: https://bugzilla.mozilla.org/show_bug.cgi?id=913070 "Bug 913070 - Move TelemetryPing to a .jsm"
[4]: https://bugzilla.mozilla.org/show_bug.cgi?id=967203 "Bug 967203 - Telemetry doesn't save pending pings on shutdown"
[5]: https://bugzilla.mozilla.org/show_bug.cgi?id=839794 "Bug 839794 - Use OS.File in Telemetry"
[6]: https://bugzilla.mozilla.org/show_bug.cgi?id=962811 "Bug 962811 - Automatic e-mail notifications of Telemetry submission rate spikes & drops"
[7]: images/telemetry_all20140214.png "Overall submission rate for the past 60 days"
[8]: http://people.mozilla.org/~mreid/telemetry/logs.aggregate.html "Aggregate Telemetry submission stats"
[9]: http://hekad.readthedocs.org "Heka"
[10]: https://bugzilla.mozilla.org/show_bug.cgi?id=960571 "Bug 960571 - switch to https for build/test downloads and hg"
[11]: https://bugzilla.mozilla.org/show_bug.cgi?id=972324 "Bug 972324 - Test for changes of the revision value in payloads"
[12]: https://bugzilla.mozilla.org/show_bug.cgi?id=972887 "Bug 972887 - Log and monitor server-side validation statistics"
[13]: https://bugzilla.mozilla.org/show_bug.cgi?id=972889 "Bug 972889 - Automatic e-mail notifications of Telemetry validation error rate spikes & drops"
[14]: https://github.com/mozilla/telemetry-server/commit/1b8c1fb963d87dcccb9849f03a3b10587938e6ab "Add support for https urls in 'revision'"
