---
layout: post
title: "From Testing to Test-First to Test-Driven"
date: 2012-05-08 11:16
comments: true
categories:
---

When I started writing tests, around 2005, I was stoked just to have the tests.

When I started writing tests first, around 2006, I was excited because I was _Doing The TDD_.

A couple of years later I found that writing tests was getting really painful.
Painful because they were so damn hard to set up, and painful because I had to wait _too damn long_ (on the order of 10-15 minutes) for the test suite to run.
I reacted to the pain by changing how I write my tests; I discovered mock objects.
My tests got faster, but they were still painful.

### A realization

In 2008 I was talking with [Corey Haines][corey] about test pain, object-oriented design, and "listening" to the former to influence the latter.
[Scott Bellware][bellware] also contributed much insight, forcing me to really think about what I hoped to gain from writing tests.

Those conversations help to crystallize it for me: the root cause of the pain was not the tests, but the design of the code under test.
I had been doing test-_first development_, not test-**driven design**.

In the years since I've honed my technique for driving design by listening to tests and I continue to seek out the ideas and experiences of other [fast test fanatics][das].

The Ruby and Rails communities have accelerated this path for many.
I would say it's not uncommon for new folks to get started where I was in 2005 or 2006.
What's more exciting is the growing numbers who are starting to feel some pain in how they test.
The next step is to become more aware of that pain; lower your pain threshold and then make it stop hurting!

### It's a journey

It was by no means an overnight endeavour.
It literally took years of work for me to figure this out, and I'm both happy and proud to say that _I'm still learning_.
I hope by putting my experience out there, yours can be better, faster, MOAR!

[corey]: http://coreyhaines.com/ "The Software Journeyman"
[bellware]: http://scottbellware.com
[das]: https://www.destroyallsoftware.com/screencasts "Screencasts for Serious Developers"
