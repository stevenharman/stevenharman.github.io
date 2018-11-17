---
layout: post
title: "A Sketch of our Ideation Pipeline"
date: 2009-01-20 11:16
comments: false
categories:
---

![Initial sketch of our Ideation Pipeline](/assets/images/posts/a-sketch-ideation-pipeline.png){:class="img-thumbnail img-thumbnail--left"}{:width="300"}
This is an initial sketch of an _Ideation Pipeline_ my team will be using to
help drive the direction of a product we’re working on. The sketch is based on
a discussion we had about how we currently get from an idea to delivering on
that idea, and how we’d like to do that going forward.

While we probably should have done a full on [Value Stream Map][2], we didn’t.
And the only excuse I have is that we’re kicking this product off so there
isn’t really a set way we do things… not yet anyhow.

At any rate, later today I’ll be turning this loose sketch into a physical
[Kanban][3] board that we’ll used to track and pull ideas through our ideation
process, and feed the resulting features into our development process.

But first, I want to explain how the whole thing will work, or at least how
we’re going to start – I’m sure we’ll tweak some things, and change out whole
parts of this process as we go along. Let’s get started.

<!-- more -->

### Starting with an Idea

Our process starts with an idea. An idea can be nearly anything that someone
_thinks_ will add value to the system. An idea might be laser-focused, or it
might be more nebulous. At this stage we don’t really care, it's only
important that we realize we have a potential value-adding idea and that we act
upon it.

The Standardized Work (designated by the blue text in the lower box of each
stage) in this stage is pretty basic, though not necessarily easy. In essence,
an idea needs to be evaluated to determine if it will deliver value.

And what does _deliver value_ mean?

> **Value is…**
>
> Seen though the eyes of those who pay for, use, and derive value from the
> systems we create.

There are many factors that go into determining if something is or is not
valuable. In our case it’s things like market demand, product vision, and
roadmap, feature parity with competitors, availability of resources, effort,
etc… Also, value is a temporal thing. So while an idea might not hold much
promise for delivering value now, that doesn’t mean that it won’t at some point
in the future.

If it’s determined that an idea will deliver value now, the idea card is moved
into the **Ready for Decomposition** state. However, if the idea won’t deliver
value now (or at least within the very short, foreseeable future), it’s moved
to a **Never** state. In the physical world that means the Idea Kanban is taken
off the board.

_But wait… if the idea has potential to provide value in the future, and you’ve
moved it to Never, won’t we lose track of it?_

YES, exactly! And you know what? We don’t care! If the idea truly has potential
for some future value, it will come back into our pipeline on its own. Why
waste time, effort, and money keeping track of a huge backlog of stuff that
might not ever happen? [DMZ that stuff][4].

### Ready for Decomposition

Right now we have no queue limits on our Ready for Decomposition queue because
we’re

  * thinking our throughput will be high enough to keep the queue short
  * going to help our customers be honest with themselves and diligent in
    determining which ideas really will provide value

However, if we find this queue to be a bottleneck, or a source of thrashing,
we’ll likely put a limit on it.

The first thing that needs to happen when any idea enters the Ready for
Decomposition state is to re-prioritize the queue. This needs to happen so the
team knows which ideas are most important and should be decomposed first.

Decomposing an idea means doing necessary analysis activities to understand and
flesh-out the idea, break it down into one or more Minimum Marketable
Feature (MMF), and defining acceptance criteria (in a loose
[Context/Specification][context-spec] style) for each feature. During the
decomposition the MMFs are transitioned to physical Kanbans that will feed into
and flow through our development pipeline. These are also known as Feature
Cards within our organization.

### When to decompose?

Idea decomposition is not a regularly scheduled activity, but is triggered by a
downstream need for Feature Cards. That is, ideas are _pulled_ through the
Ideation Pipeline by some downstream activity that will move the idea closer to
providing value. We’re pulling from [Concept to Cash][5].

It should also be noted that we only decompose as many ideas as are needed to
satisfy the source of the pull. In this case, that would be our development
pipeline’s backlog queue. If the backlog has two spots open, and we decompose
an idea into two MMFs, then we’re done decomposing for now. If we only
decompose it into one MMF, then we’ll need to decompose another idea.

And what if that second idea decomposes into two or more MMFs? Now we’ve got
more MMFs than we can fit in our backlog… oh no!

Actually, that’s just fine. Our development backlog is itself a prioritized
queue. So we just need to prioritize all of the MMFs, fill the backlog with the
most important ones, and let the rest sit in the Decomposed state until the
backlog pulls another one in.

### On to the development pipeline.

Well… that’s what those MMF Kanbans would do anyhow. But you, dear reader, will
have to wait until later – like until I write a post that covers our
development pipeline.

### Other Resources

If you’re looking for more information on some of the terms or concepts I’ve
mentioned here, be sure to check out Karl Scotland’s excellent round-up of
links/references to materials on [Kanban Software Development][6].

   [2]: http://en.wikipedia.org/wiki/Value_Stream_Mapping (Value Stream Mapping)
   [3]: http://en.wikipedia.org/wiki/Kanban (Kanban)
   [4]: http://arcware.net/dmzing-the-backlog/ (DMZing the Backlog)
   [5]: http://www.amazon.com/dp/0321437381/?tag=stevenharman-20 (Implementing Lean Software Development: From Concept to Cash)
   [6]: http://availagility.wordpress.com/kanban/ (Useful links or references to material on Kanban Software Development.)
   [context-spec]: http://stevenharman.net/toward-a-better-use-of-context-specification (Towared a Better Use of Context/Specification)
