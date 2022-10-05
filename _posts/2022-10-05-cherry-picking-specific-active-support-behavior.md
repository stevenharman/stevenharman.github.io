---
layout: post
title: "Cherry-picking Specific Active Support Behavior"
date: 2022-10-05
comments: true
tags: [rails]
image: /assets/images/posts/ruby-name-error-isolated-execution-state.jpg
description: "Cherry-picking specific extensions/behavior from Active Support."
---

The [Active Support library](as-lib) has always (or, _close enough to always_) allowed us to cherry-pick specific extensions/behaviors, to load only strictly needed dependencies.
That is, rather than loading the entirety of the library, we can load just the bits and pieces we need.
This helps keep the amount of things loaded in memory smaller, and faster by doing less work.
It can also aide in understanding by making more explicit the dependencies some code relies on.

I've been using this technique for years, and it's been solid.
Until today, while working on a Rails 7 code base, when I started seeing an error: 

```console
uninitialized constant ActiveSupport::IsolatedExecutionState (NameError)
```

As it happens, what I'd been doing for years _worked_, but more by accident than design.
Active Support 7 has _fixed the glitch_.

<!-- more -->

## How it used to "work"

Let's say we want to use Active Support's `Time` extensions for `Numeric` types.
For example, that's what that allows us to express some past date/time like this:

```ruby
5.days.ago #=> 2022-09-30 09:08:47.983955 -0400
```

We could cherry-pick in just those extensions, right in the file where we're using them, like so:

```ruby
require "active_support/core_ext/numeric/time"

5.days.ago #=> 2022-09-30 09:08:47.983955 -0400
```

But as of Active Support 7, that doesn't quite work.
It raises a `NameError` because the `ActiveSupport::IsolatedExecutionState` constant cannot be found.

```ruby
require "active_support/core_ext/numeric/time"

5.days.ago #=>
# NameError: uninitialized constant ActiveSupport::IsolatedExecutionState
# 
#       ::ActiveSupport::IsolatedExecutionState[:time_zone] || zone_default
#                      ^^^^^^^^^^^^^^^^^^^^^^^^
# from .../ruby/gems/3.1.0/gems/activesupport-7.0.4/lib/active_support/core_ext/time/zones.rb:15:in `zone'
```

## What changed?

Active Support 7 added a [new abstraction to better handle different concurrency mechanisms](as-pr-concurrency) (think, Threads vs Fibers).
In doing so, everywhere in Active Support that used to directly reference `Thread.current`, was changed to leverage `::ActiveSupport::IsolatedExecutionState`.

That means attempting to directly cherry-pick a bit of Active Support, as had been possible for years, we might also need the `IsolatedExecutionState` constant loaded.
So, the direct cherry-picking feature is… broken?

Not exactly.
It turns out, Active Support never intended us to directly cherry-pick by _only_ requiring the one file we need.
The intent, and [guides][as-guide-core-ext], have long told us to first load the library itself, and then cherry-pick any specific bits we want.
That is, we should have been doing this all along:

```ruby
require "active_support"
require "active_support/core_ext/numeric/time"

5.days.ago #=> 2022-09-30 09:08:47.983955 -0400
```

So really, nothing changed.
Before, things just kinda worked by accident.

## Does that mean all of Active Support is loaded?

I'd always _assumed_ that requiring the library would also _load_ the whole library.
Though, I should have known better.
_mea culpa_.

Instead, requiring the library via `require "active_support"` [only loads the bare minimum](as-source), while setting up all of the [Ruby `autoload`-ing](ruby-autoload) needed to reference other bits of the library later.
Huzzah!

If we do want or need to eagerly load the whole library, that's done like so:

```ruby
require "active_support/all"
```

Phew, glad to have gotten to the bottom of that one! 🎉

And a shout-out to [`@gektin`](https://github.com/gektin) for helping me dive into this one.

[as-guide-core-ext]: https://guides.rubyonrails.org/active_support_core_extensions.html#how-to-load-core-extensions "Stand-Alone Active Support"
[as-lib]: https://edgeguides.rubyonrails.org/active_support_core_extensions.html "Active Support Core Extensions"
[as-pr-concurrecy]: https://github.com/rails/rails/pull/43596 "Introduce ActiveSupport::IsolatedExecutionState for internal use"
[as-source]: https://github.com/rails/rails/blob/7-0-stable/activesupport/lib/active_support.rb#L33
[ruby-autoload]: https://medium.com/rubycademy/the-autoload-method-in-ruby-11fd079562ef "The autoload Method in Ruby"
