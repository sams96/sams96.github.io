---
title: Rgeo — 5 Years On
url: /rgeo-5-years-on
date: 2025-03-08
categories:
  - programming
tags:
  - rgeo
tootLink: "https://toot.io/@mondoman712/114126461240720151"
---

5 years ago I wrote [Rgeo](https://github.com/sams96/rgeo), a small Go package
for reverse geocoding (you input some coordinates, it gives you information
about the location). Today I am putting the projected on an indefinite hiatus.

<!--more--></br>

That's maybe a tad dramatic for my little project that I barely touch anyway,
but I thought I should be clear about the fact that I'm not really working on it
at all, and I thought I'd just write up my thoughts about it here.

</br>

In the time since, Rgeo has been noticed somewhat (much more than I expected),
gaining over 50 stars on GitHub and getting some really nice contributions. I am
very happy with how the project has been received by others, despite my neglect.

My main motivation for writing Rgeo in the first place was as a portfolio piece,
something that I could show prospective employers to prove that I can write
decent Go code, since at that time I didn't have any professional experience. It
served that purpose well, I got a job writing Go not long after the initial
release. And after that I lost the motivation to work on it, since I had a full
time job writing Go.

I have been reflecting on this project recently because, despite the fact that I
am still waiting Go professionally, I've had some more motivation to work on my
own projects, but not this project. I don't have any use for the package myself
so I’m not finding issues nor coming up with new features to add.

## What to use now

 * [smilyorg/tinygpkg](https://github.com/smilyorg/tinygpkg) - Very nice package
 inspired by Rgeo. It uses a different approach to vastly reduce startup time at
 the cost of slightly slower queries.
 * [authenticvision/rgeo](https://github.com/authenticvision/rgeo) - The most
 active fork of Rgeo, and most of the contributions I’ve had have come from
 these guys.
 * Rgeo - There’s no external API calls here so I don’t really see there being
 any security concerns, and I will try to keep dependencies up to date and be
 better at merging any fixes that come in. There just won’t be any new features
 for the time being (not that there has been for a while).

## Rgeo v2.0

I have an idea of what I would like to do for a version 2 if I ever come back to
this project, and that is to split the querying logic (which would stay in the
main package) from the parts specific to the natural earth data geojson files
used (which would go into a new sub-package). This would provide a more general
use for the package in handling data from any source while still giving easy
access to the same existing functionality. I also like
[SaTae66](https://github.com/SaTae66)’s snapping PR, but I wonder if similarly
things could be generalised and more tools could be provided, for example
querying for the closest geojson feature to a given point.
