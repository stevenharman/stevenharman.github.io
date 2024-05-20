---
layout: post
title: "On Flaky Tests, Time Precision, and Order Dependence"
date: 2024-05-20
comments: true
tags: [ruby, rails, testing]
image: /assets/images/posts/not-sure-if-flaky-test.jpg
description: "Some footguns and solutions for flaky tests due to precision of Time and order dependence."
---

A flaky test is one that fails unpredictably, without a corresponding change in the code under test.
These often show up in <abbr title="Continuous Integration">CI</abbr> runs where a test unrelated to any change made suddenly fails, and then mysteriously passes when re-run.
There are many type and causes of "flakes".
Today I want to talk about flaky tests caused by time precision, and time-order dependence.

The specific tools mentioned here will be Ruby, Rails, and PostgreSQL specific.
But analogs exist in nearly all other languages and frameworks, so the techniques are broadly applicable. 

{% include figure.html src="not-sure-if-flaky-test.jpg"
                       alt="Frye meme: Not sure if I broke something, or it's a flaky test"
                       caption="Not sure if I broke something, or it's a flaky test…" %}

<!-- more -->
## 🥶 Freeze Time

Sometimes a test assumes that time will elapse (or _not_ elapse) during the execution of the test.
In such tests, when comparing time/timestamps, try to work with entirely predictable time.
Ideally by making time… umm… more static.

In [RSpec][rspec], you can ensure that `Time.now` and friends always return the same value during a single test execution by _freezing_ time with [Rails' `freeze_time` helper][rails_freeze_time].
From the docs:

```ruby
Time.current #=> Sun, 09 Jul 2017 15:34:49 EST -05:00
freeze_time do
  sleep(1)
  Time.current #=> Sun, 09 Jul 2017 15:34:49 EST -05:00
end
Time.current #=> Sun, 09 Jul 2017 15:34:50 EST -05:00
```

I prefer to add a bit of RSpec metadata to do the magic. 🪄

```ruby
# spec/spec_helper.rb

config.around(:each, :time_frozen) do |example|
  freeze_time do
    example.call
  end
end
```

With that, you can add the `:time_frozen` metadata to a wrapping `context` or `describe` block:

```ruby
RSpec.describe MyClass, :time_frozen do
  # Time is frozen here… 🥶
end
```

Now all references to `Time.now` and friends are frozen.
Time will not move forward inside that block.

It's important to know that any setup (like RSpec `let!`, `before`, or `around` blocks) happening before the time-frozen block did not have time frozen, and so can have slightly different values `Time.now` than inside the block.
We'll come back to this later. 😉

[rails_freeze_time]: https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html#method-i-freeze_time "ActiveSupport's #freeze_time helper"
[rspec]: https://rspec.info "Behaviour Driven Development for Ruby. Making TDD Productive and Fun."

## ⏱️ Platform-specific Time Precision

Ruby's `Time` and `DateTime` objects have up to nanosecond precision.
That is, up to 9 significant digits after the decimal point.
e.g. `Time.utc(2024, 5, 20, 0, 0, 0.123456789)` would result in a `Time` object with 123,456,789 nanoseconds of precision.

However, the precision of `Time.now` (and friends) varies by platform.
On macOS, it's limited to microseconds - 6 significant digits after the decimal point.
Linux tends to be nanoseconds.

This usually doesn't matter because any single test run happens on the same machine.
However, there's a wrinkle: PostgreSQL.
Or, more generally, the precision of time-like data in your database.

PostgreSQL's `time`, `timestamp` and `interval` data types have a [resolution of microseconds][pg_time_data].
And MySQL has a similar [limitation on precision][mysql_time_data].
So during a CI run, for example, it's possible to do something like this:

```ruby
now = Time.now #=> 2024-05-20 19:58:45.660142789 +0000
BlogPost.create(published_at: now)

post = BlogPost.last
expect(post.published_at).to eq(now) #=> Flakes b/c post.published_at lost resolution

post.published_at #=> 2024-05-20 19:58:45.660142000 +0000
```

In my experience, the need for directly comparing to a time field from the database is rare.
But in those rare cases, I prefer to `#round` the in-memory time to the same precision as the database column.

```ruby
expect(post.published_at).to eq(now.round(6))
```
Another option is more fuzzy expectations/matching.
RSpec has [the `be_within` matcher][rspec_be_within], for example.
Though I find this more confusing than helpful, and tend to avoid it.

[mysql_time_data]: https://dev.mysql.com/doc/refman/8.0/en/date-and-time-type-syntax.html "MySQL Date/Time Types"
[pg_time_data]: https://www.postgresql.org/docs/current/datatype-datetime.html "PostgreSQL Date/Time Types"
[rspec_be_within]: https://www.rubydoc.info/github/rspec/rspec-expectations/RSpec%2FMatchers%3Abe_within "RSpec::Matchers#be_within"

## 🤹 Time-order Dependent Tests

In that contrived example it's easy to see how we end up with mismatched precision.
A more tricky variation occurs when we create a bunch of test data and then make assertions on that data, based on the order of various time fields.

```ruby
posts = FactoryBot.create_list(:post, :published) #=> Generate Post's with #published_at = Time.now at time of creation
sorted_posts = posts.sort_by(:published_at).reverse #=> sort the in-memory list, "newest" first
recent_posts = RecentPostQuery.call
expect(recent_posts).to eq(sorted_posts) #=> Flakes
```
In this case, the `posts` collection was held in memory, with each `Post#published_at` value having full nanosecond precision.
When saving to PostgreSQL, the precision was reduced to microsecond.
Meaning it's possible two `Post`s were created fast enough that when sub-microsecond precision is lost, they have the "same" `#published_at` in the database.
Sorting equal values is non-deterministic, so `Post` instances with the same `#published_at` can have different ordering within the `recent_posts` collection.
And so… flaky test failures.

A naive way to handle this would be to [`#reload` the posts collection][active_recored_reload] before sorting it in-memory.
Though that can result in the same problem of loss of precision, with the in-memory collection having non-deterministic ordering.

A robust way to solve this is to use explicit `Time`/`DateTime` values when creating the test data.
After all, the ordering-by-time is actually the thing we're trying to test.
Being more explicit about that in test setup helps our future selves understand which parts of setup are important, vs incidental.

```ruby
oldest = FactoryBot.create(:post, published_at: 2.days.ago)
middle = FactoryBot.create(:post, published_at: 1.day.ago)
newest = FactoryBot.create(:post, published_at: Time.now)
recent_posts = RecentPostQuery.call
expect(recent_posts).to eq([newest, middle, oldest]) #=> works every time!
```
[active_recored_reload]: https://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-reload

## 🥶⏱️ Precision of Frozen Time

Using Rails' built in `#freeze_time`, `#travel_to`, etc… will result in all sub-second precision being lost, by default.
This can be surprising, or at least unexpected.
I know it surprised me!

As with freezing time in general, this loss of precision usually doesn't matter.
However, it can cause issues similar to those above where a loss of precision leads to non-deterministic ordering.
Consider this terribly made up example:

```ruby
let!(:post) { FactoryBot.create(:post, :published) }

context "with a draft post" do
  let(:draft) { FactoryBot.create(:post, :draft) }

  before do
    freeze_time
  end

  it "includes a published draft as the newest Post" do
    draft.publish! #=> sets published_at = Time.now
    recent_posts = RecentPostsQuery.call

    expect(recent_posts).to eq([draft, post]) #=> Flakes
  end
end
```

This can be tricky to spot.

Here the `let!(:post)` happened before we froze time, so it might have a `#published_at = 2024-05-20 0:0:0.123456000 +0000`.
Then, a fraction of a second later (because computers are fast), we freeze time, create the `draft`, and `#publish!` it.
Since we know the creation and publication of the `draft` happens after the creation of `post`, this setup seems to make sense.

Except frozen time has no sub-second precision.
So it's possible that the`draft`'s `published_at = 2024-05-20 0:0:0.000000 +0000`.
Which is a fraction of a second _before_ the `post`'s, rather than after it.

We might try reaching for our `:time_frozen` RSpec helper here, relying on RSpec's behavior to ensure all `before` blocks (which include `let!`) run before the actual test code (`it` block).
And indeed that is the case.
But this is also subtly broken as now all of the times lack sub-second precision.
Meaning we're back in the non-deterministic ordering problem.

```ruby
let!(:post) { FactoryBot.create(:post, :published) }

context "with a draft post", :time_frozen do
  let(:draft) { FactoryBot.create(:post, :draft) }

  it "includes a published draft as the newest Post" do
    draft.publish! #=> sets published_at = Time.now
    recent_posts = RecentPostsQuery.call

    expect(recent_posts).to eq([draft, post]) #=> Flakes b/c time's can be equal
  end
end
```

The solution, like before, is to be explicit about time values when ordering based on them is important to the test.

### Preserving sub-second precision

With newer versions of Rails, sub-second precision can optionally be [preserved using the `with_usec` option][active_support_with_usec].
You need to opt into this at each `freeze_time`, `travel`, and `travel_to` call site, but at least it's possible!

```ruby
freeze_time(with_usec: true) do
  # Time here is frozen, and includes sub-second precision.
end
```

[active_support_with_usec]: https://github.com/rails/rails/blob/747a03ba7722b6f0a7ce42e86cea83cf07a2e6ef/activesupport/lib/active_support/testing/time_helpers.rb#L133

## 🎁 Wrapping up

In summary, take care when directly comparing or sorting on time (or time-like) values.
And when setting up test data where time is important, be explicit about the exact time values being used.

Good luck.
And happy testing!

### 🙏 Thanks

Thank you to [Hazel](https://hibachrach.com) for feedback and proofing!
