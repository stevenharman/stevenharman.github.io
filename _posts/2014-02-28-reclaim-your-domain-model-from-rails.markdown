---
layout: post
title: "Reclaim Your Domain Model from Rails"
date: 2014-02-28 14:51:47 -0500
comments: true
tags: [ruby, rails, design]
---

**TL,DR;** When building an application using Rails, I prefer to keep all my
model in `app/models/`. I reserve `lib/` for those other things - those
not-my-domain-things. I'd like to explain the *what* and *why*.

![Boundaries Amongst the Fields; Deep Greens](/assets/images/posts/deep-greens-med.jpg){:class="right"}{:width="300"}

Rails has a history of co-opting names, as happened when the `ActiveRecord`
library used the [active record pattern][active_record] name. A similar
co-opting has happened with the [MVC pattern][mvc_pattern] wherein many believe
Rails is an example of the MVC design pattern. In truth, it's probably closer
to [MVC Model 2][mvc2_pattern]... but I digress.

### Model View What's-that-now?

**MVC** stands for Model, View, Controller. In Rails-land we know what the
**Controllers** are. And while we don't have **Views** in the way that MVC
meant, we do have view-templates, and we call those our views. The **Model** is
meant to be all the things it takes to model our problem domain. As applied to
Rails, the **Model** seems the most misunderstood/misused of the MVC
triumvirate.

<!-- more -->

Rails gave us the `app/models/` directory, a natural home for our model. Yet,
that was also where Rails dumped all our `ActiveRecord::Base`-derived objects.
Combined with the "Skinny Controller, Fat Model" mantra of 2006-ish Rails-land,
that directory had grown full of bloated, tightly coupled, low cohesion
`ActiveRecored::Base`-derived objects. *Our model!*

## Models Too Fat?

Eventually we realized that shoving all those disparate concerns into a single
class was a Bad Idea™. There was a movement afoot to thin down our now
overly-fat models. So, we re-learned [some forgotten lessons][pojo] and the use
of POROs became a Good Thing™. We needed to break our domain models down into
smaller, more cohesive objects, but we had a new problem. *Where to put these
new files?*

As Rails had *reserved* `app/models/` directory for
`ActiveRecord::Base`-derived objects, we had to find a new home for those
POROs. As luck would have it, Rails had given us a `lib/` directory, and that
thing was basically deserted. We began breaking up our model and littering it
amongst the `app/models/` and `lib/` directories.

Our model had become more loosely coupled, with more cohesive parts, but it
wasn't living together. Why are we forcibly segregating the
`ActiveRecord::Base` portions from the rest of our model? If it's a single
domain model, why doesn't it live together?

These days I put all the domain-specific objects into the `app/models/`
directory. I reserve the `lib/` directory for things which could ostensibly be
[gemified][rubygems].

## So. Many. Files. So Many Boundaries.

I've heard grumblings that having all the domain model living together
results in a lot of files in the `app/models/` directory, making it hard to
organize and navigate. To this I say, *YES, it does.*

However, I suspect that is a sign of another problem. To wit, *do you actually
have multiple domains emerging, which could perhaps be broken out into discrete
apps?* If that is the case, make that boundary explicit and break out a new
application!

So long as those other domains are small or emerging it seems excessive to
split them out into their own apps. So, what do we do? Why we segregate them
with internal [domain boundaries][bounded_context], of course!

### An Example Boundary.

A lot of apps have a search feature (or set of search features).
Search is a whole domain on its own, and often not germane to the primary
domain. Pull all search concerns into a namespace, `Search::`. To help with
organization, put all files to do with search in a directory of their own. For example,
`app/models/search/`, `app/controllers/search/`, `app/views/search/`, etc...

With just a few changes we've reclaimed the `app/models/` directory, better
organized our files, and defined some new boundaries within our code base. Ah,
much better.

<small>
[*Deep Greens* image][deep_greens_image] courtesy of Brett Jordan.
</small>

[active_record]: http://www.martinfowler.com/eaaCatalog/activeRecord.html "Active Record"
[bounded_context]: http://martinfowler.com/bliki/BoundedContext.html "Bounded Context"
[deep_greens_image]: http://www.flickr.com/photos/x1brett/8025844399/ "Deep Greens - Aerial07"
[mvc2_pattern]: http://en.wikipedia.org/wiki/Model_2 "MVC Model 2"
[mvc_pattern]: http://c2.com/cgi/wiki?ModelViewController "Model View Controller"
[pojo]: http://en.wikipedia.org/wiki/Plain_Old_Java_Object "Plain Old Java Object"
[rubygems]: http://rubygems.org/ "A package manager for Ruby"
