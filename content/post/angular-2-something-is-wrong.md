+++
draft = false
title = "Angular 2. Something is wrong."
date = "2017-05-24T15:13:01+01:00"
tags = ["Angular2", "AngularJs]
categories = ["AngularJs", "FrontEnd"]
series = []
images = []
description = ""
summary = """
When something reeks, it's not a good sign.
"""

+++

It's been a few months of Angular (I have to stop using the suffix 2, because we
now have version 4)... And I have to say that I started to love TypeScript
and start to like Angular.

I had a very difficult beginning, but I could say that productivity goes up,
so I'm not mad about Angular anymore, but still dislike many things!

I had many problems in the past with AngularJS too, in particular with
angular-ui team, dependency injection and build steps.
I submitted many issues, none of them were solved, mostly because I usually provide
a workaround...

I had found a few bugs in Angular that I didn't report because already have a
workaround (I know, my bad, always report).

Sometimes you face the inevitable: there is no workaround, so you are forced
to submit an issue and provide proof (a plukr), and then I noticed
something very strange: 1298 open issues.
**WTF**! That number is huge! so I spend 10 minutes comparing a few projects.


|Project  |Open issues|Closed issues|Commits|Pull requests|SLOC   |
|---------|-----------|-------------|-------|-------------|-------|
|Angular  |1298       |9167         |7635   |204/6230     |357122 |
|AngularJS|578        |7918         |8495   |175/7301     |484732 |
|Node.JS  |738        |4498         |17561  |308/7602     |5843751|
|Julia    |1889       |9974         |37018  |483/9690     |248676 |



Conclusion.

First Angular is much bigger than AngularJS (+35%). It has the same issues
in less than half time, maybe it's a world record (who knows!). It's very
close to AngularJs current PRs so it seems very healthy. But the problem is
`Open issues`, for me it proves the Angular instability also proves
that I shouldn't lose my time reporting if I found a workaround fast.
Also, means that Angular don't triage properly.

I also did a small comparison between two new languages Node and Julia.
In the end, just by the numbers, node.js did a better job to keep issues low,
but maybe it's not fair because Node.Js change repo when leaved Joyent.
