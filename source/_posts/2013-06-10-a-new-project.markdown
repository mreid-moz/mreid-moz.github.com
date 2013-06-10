---
layout: post
title: "A New Project"
date: 2013-06-10 14:32
comments: true
categories: Mozilla
---

Over the coming weeks I'll be embarking on a new project to revamp the acquisition, storage, and processing of [Mozilla Telemetry](https://wiki.mozilla.org/Telemetry) data.

The general pipeline for Telemetry data looks like this:

1.  A Firefox user enables Telemetry in their browser
2.  The browser generates performance data as it is used
3.  Once a day, the browser submits the performance data to Mozilla via HTTPS
4.  Data is accepted by [the server](https://wiki.mozilla.org/Telemetry) and saved to a queue
5.  The queue is polled for changes and payloads are saved to persistent storage
6.  Analysis jobs are run against the persisted data, including daily aggregations
7.  Results of analysis are visualized in [Telemetry Dashboards](https://wiki.mozilla.org/Telemetry)


My initial focus will be on steps 5 and 6, specifically on improving the persistent storage format to be more space-efficient and to eliminate the need for re-processing to slice the data (by factors like Channel, Version, Day, etc).

I plan to post a link to the code repository here (as soon as there is something useful to share). 
