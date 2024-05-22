---
layout: post
title: "The No Man's Land of Web Development"
date: 2013-12-20 14:23
comments: true
tags: [rails, javascript]
---
I think about the current state of web development experiences as a continuum.
On one end we have *traditional* Rails-era web apps - full page loads, with
bits of dynamism haphazardly mixed in. On the other are rich client-side
JavaScript apps with their own structure and life cycles, standing alone and/or
talking to an HTTP API.

![No Man's Land Flanders Field France 1919.](/assets/images/posts/no-mans-land-flanders-field.jpg){:class="img-thumbnail img-thumbnail--center"}

Rails-era web apps have some great tooling and deliver a pretty nice
development experience. The shift toward rich client-side web experiences has
lead to some great tooling that makes for a [1st-class web development experience][web-dev-lineman].

## A no man's land

Between these two approaches lies a no man's land. The tooling and techniques
are focused largely on either end of the continuum despite the large population
of apps living in the middle.

<!-- more -->

For too long we've resorted to sprinkling JavaScript onto our Rails-era web
apps in an attempt to inch toward the rich and dynamic experiences we see at
the other end of the spectrum. However, not every application needs a full
rich-client experience. Often a Rails-era web app is *mostly* enough, except we
*require* some dynamism in portions of the application. But sprinkling on more
JavaScript only leads to larger tangles of costly and inflexible code. What we
really desire is that rich-client 1st-class web development experience paired with our
1st-class Rails-era web app development experience.

## For example

[Rails][rails] is a tool I use regularly, and it's often the first tool I reach
for when building those Rails-era web apps. My friends at [Test Double][testdouble]
have made great contributions to creating a great web development experience
for fat-client JavaScript applications with [Lineman][lineman], especially when
paired with frameworks like [AngularJS][angularjs] or [Ember][ember]. What I long
for, and believe is both needed and achievable, is a happy union of these two
approaches with tooling that supports them working together.

Sadly I don't have an answer for you. At least not today. But I know that
folks are working to enrich the middle ground, to reduce the frustration and
friction felt when using tools like Rails while also striving to build sensible
and coherent fat-client JavaScript apps with tools like AngularJS and Lineman.

If you're working to improve the middle ground, or you find yourself living
there, please share your thoughts, experiences, and advice so that we all might
learn and improve, together.

[angularjs]: http://angularjs.org/ "HTML enhanced web apps!"
[ember]: http://emberjs.com/ "A framework for creating ambitious web applications."
[lineman]: http://linemanjs.com/ "Lineman takes everything you love about building server-side applications so that you can find joy in your client-side applications"
[rails]: http://rubyonrails.org/ "Web development that doesn't hurt"
[testdouble]: http://testdouble.com "Custom Software Development | Columbus Ohio"
[web-dev-lineman]: http://blog.testdouble.com/posts/2013-11-12-1st-class-web-development-with-lineman.html "1st-class web development with Lineman."
