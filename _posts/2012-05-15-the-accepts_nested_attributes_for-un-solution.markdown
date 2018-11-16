---
layout: post
title: "The accepts_nested_attributes_for Un-Solution"
date: 2012-05-15 21:50
comments: true
categories:
---

![Merry-Go-Round](/assets/images/posts/merry-go-round.png){:class="right"}

Why does Rails' `accepts_nested_attributes_for` approach always feel so darned
*wrong*?

My suspicion is that it feels wrong because it likely is wrong - or at least it
is likely the wrong tool for the job.

### An example

An accounting of a recent ride I took into the `accepts_nested_attributes_for`
merry go round.

> In order to create a Message, I have to first create the Conversation it's a
> part of.
>
> This sounds like an explicit workflow, so I'll model it that way.
>
> _... hours later ..._
>
> FFFUUU, this is too complicated! There must be an easier, more Rails-y way.
>
> I know, Conversations can `accepts_nested_attributes_for :messages`. Brilliant!
>
> _... hours later ..._
>
> FFFUUU, this is too complicated! There must be a simpler way.
>
> I know, rather than shoehorning this into an overly-coupled mess I'll model
> it as an explicit workflow!
>
> **FML**.


Remember, [easy is not the same thing as simple][hickey].

_image via: [lolitanie.com][lolitanie]_

  [hickey]: http://www.confreaks.com/videos/860-railsconf2012-keynote-simplicity-matters "Simplicity Matters."
  [lolitanie]: http://blog.lolitanie.com/index.php/archives/2006/01/29/oh-oh-oh-merry-go-round/
