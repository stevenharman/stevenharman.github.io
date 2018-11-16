---
layout: post
title: "Bag of Methods Module and Grep-driven Development"
date: 2012-10-02 10:34
comments: true
categories:
---
Always looking for more concise ways to express ideas, I'd like to present two
terms for your consideration.

<dl>
  <dt><a id='bomm'>BOMM</a> <em>|bäm|</em> <i>(abbreviation)</i></dt>
  <dd>Bag of Methods Module</dd>
  <dd>An anti-pattern for "sharing behavior" or "separating concerns" of an
  object. In practice such modules often contain code that is related in name or
  function, but lacking a cohesive purpose. See also: <em><a href='#gdd'
  title='Grep-driven Development'>GDD</a></em>.
  </dd>

  <dt><a id='gdd'>GDD</a> <i>(initialism)</i></dt>
  <dd>Grep-driven Development</dd>
  <dd>A software development process that relies on
  searching full source code to find usages of methods and deduce intended
  behavior of a piece of code. Often caused by lack of coherent and cohesive
  design. See also: <em><a href='#bomm' title='Bag of Methods
  Module'>BOMM</a></em>.
  </dd>
</dl>

Thanks to fellow [Highgroover][highgroove], [Andy Lindeman][andy], for helping me
to finally define these terms. Or at least, refine them.

[highgroove]: http://highgroove.com Ruby and Ruby on Rails Consulting, Development, and Training
[andy]: http://www.andylindeman.com
