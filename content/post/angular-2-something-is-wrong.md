+++
draft = false
title = "Angular 2. Something is wrong."
date = "2017-05-24T15:13:01+01:00"
tags = ["Angular2", "AngularJs"]
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


<table class="table table-striped">
<thead>
<tr>
<th>Project</th>
<th>Open issues</th>
<th>Closed issues</th>
<th>Commits</th>
<th>Pull requests</th>
<th>SLOC</th>
</tr>
</thead>

<tbody>
<tr>
<td>Angular</td>
<td>1298</td>
<td>9167</td>
<td>7635</td>
<td><sup>204</sup>⁄<sub>6230</sub></td>
<td>357122</td>
</tr>

<tr>
<td>AngularJS</td>
<td>578</td>
<td>7918</td>
<td>8495</td>
<td><sup>175</sup>⁄<sub>7301</sub></td>
<td>484732</td>
</tr>

<tr>
<td>Node.JS</td>
<td>738</td>
<td>4498</td>
<td>17561</td>
<td><sup>308</sup>⁄<sub>7602</sub></td>
<td>5843751</td>
</tr>

<tr>
<td>Julia</td>
<td>1889</td>
<td>9974</td>
<td>37018</td>
<td><sup>483</sup>⁄<sub>9690</sub></td>
<td>248676</td>
</tr>
</tbody>
</table>



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
