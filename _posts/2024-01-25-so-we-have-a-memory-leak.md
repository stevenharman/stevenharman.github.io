---
layout: post
title: "So We've Got a Memory Leak…"
date: 2024-01-25
comments: true
tags: [ruby, rails, heroku]
image: /assets/images/posts/memory_leak/initial-memory-spikes.png
description: "A story of finding, understanding, and fixing a memory leak in our Rails app. With graphs and code!"
---

Memory leaks happen.
And if you're here, reading this, I'd bet you're dealing with one.
First things first - are you [sure it's a leak, and not bloat][med_tracking_2021]?

Okay, so it's a leak.
Much has been written about various tools for [profiling a leak][appsignal_dive], [understanding heap dumps][cloudbees_guide], [common causes of leaks][devops_understanding], [work being done to improve Ruby's memory layout][shopify_variable], and so much more.
Ben Sheldon's recent "[The answer is in your heap: debugging a big memory increase in Ruby on Rails][island94]" is a great example.
Ben mentions and shows how he and some teammates used several different tools to generate heap dumps, analyze and interrogate them, and ultimately find and fix the source of a leak in Rails itself.

These are all great resources, and can be crucial knowledge in our hunt for a leak.
But they all suppose we already have an idea of where to start looking, or use stripped down examples to showcase the tools.
The [`memory_profiler` Gem][memory_profiler] can profile memory allocations around some suspect code, if we already know that code is suspect.
[Derailed Benchmarks][derailed] can get us
started on tracking down a leak in the overall stack, or a known problematic resource, and generate heap dumps for us.
Comparing those dumps with [heapy][] can point us in the right direction by revealing memory being unexpectedly retained over time.
We can use [sheap][] to track down exactly where problematic objects allocations are happening, once we've identified those problematic objects.

But what if we've reviewed all recent code changes, and nothing stands out?
Or if the leaks don't happen consistently, across all instances of the running app?
Or memory starts leaking different times?
Where do we even start?

<!-- more -->

That's what I was asking myself recently with a decade old Rails app I work on.
This is a real production app, driving substantial revenue.
It serves a sustained load of 400-500 requests/second, with peaks into several thousands/second.

Then one, nondescript day, amongst the usual flow of deploys, memory started spiking. 📈
The pager started yelling. 📟
And we had what looked like a memory leak. 💧

<figure>
  <img src="/assets/images/posts/memory_leak/initial-memory-spikes.png"
       alt="Graph of memory starting to spike over and over mid-way across the view"
       title="We have a memory leak! So. Many. Spikeys!">
  <figcaption class="text-center">We have a memory leak! So. Many. Spikeys!</figcaption>
</figure>

[sheap]: https://github.com/jhawthorn/sheap "sheap - A library for interactively exploring Ruby Heap dumps. Sheap contains contains a command-line tool and a library for use in IRB."
[heapy]: https://github.com/zombocom/heapy "Heapy (Ruby Heap Dump Inspector) - A CLI for analyzing Ruby Heap dumps."
[derailed]: https://github.com/zombocom/derailed_benchmarks/blob/main/README.md#i-want-more-heap-dumps "Derailed Benchmarks - A series of things you can use to benchmark a Rails or Ruby app."
[memory_profiler]: https://github.com/SamSaffron/memory_profiler "MemoryProfiler - A memory profiler for Ruby"
[island94]: https://island94.org/2024/01/the-answer-is-in-your-heap-debugging-big-rails-memory "The answer is in your heap: debugging a big memory increase in Ruby on Rails"
[med_tracking_2021]: https://medium.com/klaxit-techblog/tracking-a-ruby-memory-leak-in-2021-9eb56575f731#5051 "Tracking a Ruby memory leak in 2021"
[cloudbees_guide]: https://www.cloudbees.com/blog/the-definitive-guide-to-ruby-heap-dumps-part-i "The Definitive Guide to Ruby Heap Dumps, Part I"
[appsignal_dive]: https://blog.appsignal.com/2022/08/10/a-deep-dive-into-memory-leaks-in-ruby.html "A Deep Dive into Memory Leaks in Ruby"
[devops_understanding]: https://blog.devops.dev/understanding-memory-leaks-in-ruby-on-rails-applications-43a84222f49c "Understanding Memory Leaks in Ruby on Rails Applications"
[shopify_variable]: https://shopify.engineering/ruby-variable-width-allocation "Optimizing Ruby's Memory Layout: Variable Width Allocation"


## 🔥 First put out the fire {#put-out-the-fire}

It certainly looked like a leak, rather than bloat, meaning we had one sure-fire way to temporarily fix the problem.
Restart things!
We run on [Heroku][], and what we're seeing above are per-[Dyno][] memory numbers.
In the short term, we'd either let our normal cadence of many-deploys-a-day restart our `web` instances, or we'd manually restart individual Dynos approaching their memory limit.

[heroku]: https://heroku.com "Heroku is a container-based cloud Platform as a Service (PaaS)."
[dyno]: https://www.heroku.com/dynos "Dynos: the heart of the Heroku platform"

## 🔎 Look for suspect changes {#look-for-suspect-changes}

With the fire under control we turned our attention to changes that happened in the hours before the first big spike.
The graph above doesn't show our deploy markers (they were too distracting at this granularity) but starting with the deploy just inside of the first spike and working backward, we audited all code changes going back three days.

We found a few changes that seemed like they *could* be related.
One of them introduced a memory leak in `development` mode due to Rails' code reloading.
Another caused us to hit Redis more than we intended during certain request filtering.
The third looked like we were making more database calls, loading more `ActiveRecord` instances than we wanted (a classic N+1).
We fixed the first two and rolled back the third, deploying them one at a time.
The memory leak continued. 📈

No other code changes *looked* like they could cause a leak.
But, we had made a few tooling changes to start collecting Ruby language and Puma pool usage metrics, so we backed those out and deployed.
The memory leak continued. 📈

## 🧩 Look for patterns {#look-for-patterns}

Next we went looking for patterns to the memory growth.
Of note, it was only our `web` Dynos that were leaking.
All of our [Sidekiq][] and [Delayed::Job][delayed_job] (yes, both, for… reasons lost to history) Dynos looked just fine.
Also, not all `web` Dynos were always leaking memory.
Sometimes they'd go hours with relatively flat-ish memory usage consistent with a long-running web process.
Then one, or a few, or all of them would start to leak memory.
This made us suspicious that perhaps the leak was specific to a certain *kind* of traffic, rather than a problem in the stack, or with volume of traffic.

<figure>
  <img src="/assets/images/posts/memory_leak/memory-spikes-delayed.png"
       alt="Graph of memory flat for 4 hours, and then starting to rise."
       title="Low-traffic period, just post deploy. Hours pass before 2 of 4 Dynos start to leak memory.">
  <figcaption class="text-center">Low-traffic period, just post deploy. Hours pass before 2 of 4 Dynos start to leak memory.</figcaption>
</figure>

When we looked at our Puma workers, we also saw that not all workers on a single Dyno were leaking.
We run Puma in clustered mode, with 12 worker processes for the 8 <abbr title="Virtual CPU">vCPU</abbr> on each Dyno.
It wasn't unusual to see just a couple of the 12 processes were using up nearly all of the memory on the Dyno.
i.e., a couple workers were leaking, while the rest were fine.

Due to aggressive sampling we were unable to use our [OpenTelmetry Traces][otel_traces] to match specific kinds of traffic to specific Dynos at times that correlated to the start of the leaks.
But we still had a strong suspicion the leaks had *something* to do with particular traffic.
One of the smart folks I work with had the idea to try correlating via our logs - which are not sampled.
But the tooling there didn't make that particularly easy.
So we put that idea in our back pocket and moved on to another.

[otel_traces]: https://opentelemetry.io/docs/concepts/signals/traces/ "OpenTelemetry Traces"
[sidekiq]: https://sidekiq.org/products/enterprise.html "Simple, efficient background jobs for Ruby."
[delayed_job]: https://github.com/collectiveidea/delayed_job "Delayed::Job (or DJ) encapsulates the common pattern of asynchronously executing longer tasks in the background."

## 🚛 Go to the dumps! {#go-to-the-dumps}

If memory is being leaked, looking at dumps of Ruby's memory heap is probably a good idea.
After all, all of those very smart Internet Folks wrote lots of words about it.

### Step 0: Get the tooling in place {#step-0-tooling}

We use [`rbtrace`][rbtrace] to attach to a running Ruby process, meaning it needs to be loaded into the running process.
Basically, make sure it's in the `Gemfile`.
It's generally fine to have this loaded in production.
But we've made it somewhat toggle-able; we can opt in/out of loading `rbtrace` via an Environment Variable:

```ruby
gem "rbtrace", require: String(ENV.fetch("FEATURE_ENABLE_MEMORY_DUMPS", false)) == "true"
```

### Step 1: Find a leaking process {#step-1-find-leaking-process}

Running on Heroku meant we needed to dump the heap of a live, running `web` process, from within a Dyno itself.
Luckily, Herkou and the Ruby community have already built all of the tooling necessary.

Using [`heroku ps:exec`][heroku_ps_exec] to open an SSH tunnel into a leaking Dyno, we needed to find a particular Puma worker process to dump.
We went to our old *nix friend `ps`.

```bash
ps -eo pid,ppid,comm,rss,vsz --sort -rss | grep ruby
```

This gave us a list of all of the Ruby processes running on the Dyno, sorted by their [RSS][] (amount of memory being used by the process).
In the case of a `web` Dyno, most of them have the same <abbr title="Parent Process IDentifier">`PPID`</abbr>.
These are the Puma workers.
The process with a matching <abbr title="Process IDentifier">`PID`</abbr> is the Puma primary (master) process.
It will look something similar for Sidekiq.
Other Ruby tools that run only a single process would look a little different.
Picking the process with the most memory (i.e., the leakiest of them) we use its `PID` to and `rbtrace` to start tracing memory allocations.

### Step 2: Enable memory allocation tracing {#step-2-enable-tracing}

We `export` that worker process ID as `DUMP_PID` to be used as a variable in later steps.
This helps prevent silly typos and much face-into-keyboard-ing later on.

```bash
export DUMP_PID=<pid>
# Turn on allocation tracking in the Ruby process.
# This will impact performance; it can use a lot of memory and CPU
rbtrace -p "$DUMP_PID" -e "Thread.new{require 'objspace';ObjectSpace.trace_object_allocations_start}.join"
```

### Step 3: Dump the heap {#step-3-dump-the-heap}

```bash
# Generate the first heap dump
rbtrace -p "${DUMP_PID}" -e "Thread.new{require 'objspace'; GC.start(); io=File.open('/tmp/heap-${DUMP_PID}.json', 'w'); ObjectSpace.dump_all(output: io); io.close}.join" --timeout=600
```

The `.json` file can be quite large.
On a leaking process that had been running for a few hours, it was 5-6<abbr title="Gibibyte">GiB</abbr> in size.
I'd recommend `gzip`-ing it.

```bash
gzip "/tmp/heap-${DUMP_PID}.json"
```

That will replace the `.json` dump file with a `.json.gz` file which could be up to an order of magnitude smaller.

### Step 4: Fetch the dump {#step-4-fetch-the-dump}

This will differ depending on where the application is deployed, but on Heroku we run the following (adjusting for exact file and Dyno names) from a shell on our local machine.

```bash
heroku ps:copy /tmp/heap-<pid>.json.gz --dyno=<web.1> --app=<app name>
```

Decompress and move the file somewhere organized.
I'd recommend renaming it based on which dump this is; we'll grab at least three if we're going to use `heapy` to try to find retained memory, for example.

```bash
mv heap-<pid>.json.gz ~/dumps/.
gzip --keep --decompress ~/dumps/heap-<pid>.json.gz
```

Repeat steps 3 and 4 to grab as many dumps as needed.

### Step 5: Clean up after ourselves {#step-5-clean-up}

Be sure to turn off the allocation tracking, and probably remove those dumps from the Dyno.
Or just restart the Dyno.

```bash
rbtrace -p "$DUMP_PID" -e "Thread.new{GC.start;require 'objspace';ObjectSpace.trace_object_allocations_stop;ObjectSpace.trace_object_allocations_clear}.join"
```

[heroku_ps_exec]: https://devcenter.heroku.com/articles/exec "Heroku Exec (SSH Tunneling)"
[rbtrace]: https://github.com/tmm1/rbtrace "rbtrace: like strace, but for ruby code"
[rss]: https://stackoverflow.com/questions/7880784/what-is-rss-and-vsz-in-linux-memory-management "What is RSS and VSZ in Linux memory management"

## 🧐 Analyze the heap dumps {#analyze-the-heap-dumps}

We tried looking at the retained memory reports via `heapy`, and meandering the diffs via `sheap`, but with no particular spot to start looking, it felt like looking for a specific needle in a stack of needles.
(Thanks for that line, Tom Hanks.)

In hopes of *seeing* something that looked off, [I went in search][ruby_social] of a way to visualize the heap dumps.
And The Internet™ did not disappoint!
[`reap` is a tool for analyzing a Ruby heap dump's reference graph][reap], and generating visualizations.
Including flame graphs!
Here's a graphs of the third dump we took off of a leaking process, per the steps above.

```bash
reap path/to/heap-<pid>-2.json --count=15 --flamegraph=dump-2.svg
```
The way to read this is starting at the top and moving downward are references from a "root" (in the view of the Ruby Garbage Collector) to other objects.
That is, objects further down are references by objects above them.
And the more memory an object (or an object it references) is holding onto, the wider the cell in the graph.

<figure>
  <img src="/assets/images/posts/memory_leak/flamegraph-2.png"
       alt="Flame graph showing a Thread holding 1.9GiB of memory"
       title="Flame graph showing a Thread holding 1.9GiB of memory">
  <figcaption class="text-center">Flame graph showing a Thread holding 1.9GiB of memory</figcaption>
</figure>

Notice anything odd?
What's up with that `Thread` holding 1.9GiB of memory?
Or really, it's the `Array` at the bottom, holding onto references to `32,067` other objects, in turn holding 1.9GiB of memory.
So what the heck is that array?
Where did it come from?
Why is it referencing so much memory?

<figure>
  <img src="/assets/images/posts/memory_leak/flamegraph-2-annotated.png"
       alt="Flame graph calling out an Array of 32k things, referencing 1.9GiB of memory"
       title="Flame graph calling out an Array of 32k things, referencing 1.9GiB of memory">
  <figcaption class="text-center">Flame graph calling out an Array of 32k things, referencing 1.9GiB of memory</figcaption>
</figure>


[ruby_social]: https://ruby.social/@stevenharman/111791414166558864 "Me, asking questions on ruby.social"
[reap]: https://github.com/oxidize-rb/reap "A tool for parsing Ruby heap dumps by analyzing the reference graph."

## 🧗 Go spelunking with `sheap` {#go-spelunking-with-sheap}

Using the latest `main` branch from [`sheap`][sheap] (because some of the features like `find_path` weren't yet released to RubyGems), we compared the second and third dumps.
It will take a while to parse these dumps, which were pushing 6GiB in our case. ☕️


```bash
~/code/jhawthorn/sheap/bin/sheap heap-<pid>-1.json heap-<pid>-2.json
```

Opens us into an <abbr title="Interactive Ruby">IRB</abbr> console where we can explore the dumps and the diff between them.
See the `README` (and the source code) for all that it can do.

```ruby
irb: t = $after.at("0x55adecb68498")
===> <DATA 0x55adecb68498 VM/thread (137 refs)>
irb: $after.find_path(t)
===> [<ROOT vm (4014 refs)>, <DATA 0x55addb8deff8 ractor (31 refs)>, <DATA 0x55adecb68498 VM/thread (137 refs)>]
```

We first investigated that `Thread` to understand where it was coming from.
Our suspicion was that it was a background thread from one of our telemetry or metrics tools - maybe the <abbr title="OpenTelemetry">OTel</abbr> tracer, or error reporter, or something like that?
But, `sheap` told us it was a Puma thread - the one actually working requests, meaning it's "the Rails thread." So that question is answered.

Next we wanted to know more about that `ActiveSupport::SubscriberQueueRegistry` object.
We went looking at the stable branch for the version of Rails we're currently on (6.1) and found that [it's basically per-`Thread` `Hash` used to store lists of `ActiveSupport::Subscriber` instances][rails_L155], based on the event name.
And `ActiveSupport::Notifications::Event` objects are pushed onto and popped off that per-thread, per-event-name list (`Array`) as `ActiveSupport::Notifications.instrument` blocks are run.
Going back to `sheap` we can see that in action - the `SubscriberQueueRegistry` instance holds a reference to a `Hash`, which has references to some `String` and `Array` objects.

```ruby
irb: asr = $after.at("0x55adeeb31798")
===> <OBJECT 0x55adeeb31798 ActiveSupport::SubscriberQueueRegistry (1 refs)>
irb: asr.references
===> [<HASH 0x55adeeb31748 (6 refs)>]
irb: asr.references.first.references
===>
[<STRING 0x55adeb01afb0 "ActiveRecord::LogSubscriber-9300">,
 <ARRAY 0x55adeeb314f0>,
 <STRING 0x55adf09ffaf8 "Lograge::LogSubscribers::ActionController-276520">,
 <ARRAY 0x55adf09ffb20 (1 refs)>,
 <STRING 0x55adf4a029e8 "Lograge::LogSubscribers::GRPCWeb-276540">,
 <ARRAY 0x55adf4a02a10>]
```

The only `Array` with anything in it was in the middle of the flame graph with addresses ending `ffb20`.
It held a reference to a single `ActiveSupport::Notifications::Event` instance.
And that instance then held a reference to the `Array` of 32k+ other objects.
If we look at the code for that `Event` class, we [see it contains a single `@children` ivar `Array`][rails_L64].
So, it's an `Array` of what then?
*Child* `Event` instances, presumably.
We grabbed a couple to confirm.

```ruby
irb: event = asr.references.first.references[3].references.first
===> <OBJECT 0x55adf4fdda70 ActiveSupport::Notifications::Event (4 refs)>
irb: children_ary = event.references.last
===> <ARRAY 0x55adf4fdda20 (32067 refs)>
irb: children_ary.references.first
===> <OBJECT 0x55adf523e9e0 ActiveSupport::Notifications::Event (4 refs)>
irb: children_ary.references.first.references
===> [<STRING 0x55addbe19ae0 "redirect_to.action_controller">, <HASH 0x55adf523e9b8 (10 refs)>, <STRING 0x55adedec2188 "19b24a9862f0e692859e">, <ARRAY 0x55adf523e990>]
```

Sure enough, the huge `Array` is a list of "children" `Event` objects.
This first child `Event` was for a `redirect_to.action_controller` event name.
And it would have been pushed onto this `Event#children` list because some [other `Event` was already on the `event_stack`][rails_L138].

At this point our intuition was telling me something was wrong here - these `Event`s were incorrectly being put on this `#children` `Array`.
We wanted to know what else that child `Event` had to tell us, like what is in that `Hash` it's holding a reference to?

```ruby
irb: first_child = children_ary.references.first
===> <OBJECT 0x55adf523e9e0 ActiveSupport::Notifications::Event (4 refs)>
irb: first_child.references[1].references
===>
[<OBJECT 0x55adf4e54e38 ActionDispatch::Request (10 refs)>,
 <STRING 0x55ade7965bf0 "PresentationController">,
 <STRING 0x55adf4250150 "view">,
 <HASH 0x55adf4f6d4f0 (507 refs)>,
 <OBJECT 0x55adf4f5c998 ActionDispatch::Http::Headers (1 refs)>,
 <STRING 0x55adf4cccca0 "GET">,
 <STRING 0x55adf4f6e8a0 nil (1 refs)>,
 <HASH 0x55adf5378c70 (1 refs)>,
 <ARRAY 0x55adf5378ae0 (2 refs)>,
 <OBJECT 0x55adf5382310 (0x55adf5400058) (4 refs)>]
```

Whoa, an entire `ActionDispatch::Request` object. 😱
The request that was being redirected.
What did that look like?
Maybe we could reproduce this locally?

```ruby
irb: first_child.references[1].references.first
===> <OBJECT 0x55adf4e54e38 ActionDispatch::Request (10 refs)>
irb: first_child.references[1].references.first.references
===>
[<HASH 0x55adf4ccced0 (222 refs)>,
 <HASH 0x55adf4f6d4f0 (507 refs)>,
 <STRING 0x55adf4f6e8f0 "/view/abc123def456?parked=junk&parking=junk&parseSchema=junk&… eliding more param tampering … &password=[FILTERED]& … &portbl=junk">,
 <STRING 0x55addc723e60 "https://">,
 <STRING 0x55adf4cccca0 "GET">,
 … # eliding more stuff
 <OBJECT 0x55adf4f5c998 ActionDispatch::Http::Headers (1 refs)>]
```

Well look at that.
It‘s a mostly valid request, to an actual `Route` in the app, with a valid public ID of a resource (changed to protect the innocent, of course).
What's not legit is the parameter tampering - the work of a scanner very likely.
What especially stood out was the `password=[FILTERED]` , which means something was catching the request and cleaning sensitive info we'd not want in logs and such.

[rails_L64]: https://github.com/rails/rails/blob/517ff4b7c6c6aeaa588993475277d816b0ba038e/activesupport/lib/active_support/notifications/instrumenter.rb#L64
[rails_L138]: https://github.com/rails/rails/blob/517ff4b7c6c6aeaa588993475277d816b0ba038e/activesupport/lib/active_support/subscriber.rb#L138-L139
[rails_L155]: https://github.com/rails/rails/blob/517ff4b7c6c6aeaa588993475277d816b0ba038e/activesupport/lib/active_support/subscriber.rb#L155

## 🔄 Reproduce it {#reproduce-it}

With the questionable looking query path and params, we opened the production app in an incognito browser window and made a request.
We were met with a <abbr title="Internal Server Error">500</abbr> server error.

🎉 Success! ✨

Logs confirmed that it was an error.
Specifically a `URI::InvalidURIError`.
The logs also recorded which Dyno the request had hit.
We popped open our memory graphs and by sheer luck, that Dyno was currently showing normal memory usage.
We put a temporary deploy freeze in place so we could monitor memory usage for a few minutes and before long the trend line was clear.
We had a leak.

With some confidence we were leaking `Event` objects as part of the `ActiveSupport::Notifications` tooling, we still wanted to reproduce it locally so we could step through the code and see what was going on.
Leaning on a little bit of `binding.pry` and even more `puts` debugging in the `activesupport` Gem (if you don't know, `bundle open <gem name>`, [check it out](https://bundler.io/v2.5/man/bundle-open.1.html)), we eventually reproduced the situation locally.
And with a backtrace!

The error's backtrace pointed to the `uri` Gem (part of Ruby's standard library), which was being [used by Bugsnag's `Bugsnag.cleaner.clean_url` method][bugsnag_L45], as part of a `ActiveSupport::Notifications.subscribe` block.
Which sounded awfully familiar.
A little more debugging and stepping through the Bugsnag and `ActiveSupport` code and We'd narrowed in on the problem(s).

<figure>
  <img src="/assets/images/posts/memory_leak/puts-debugging-before.png"
       alt="Logs showing a Event object left on the stack after the error is raised"
       title="Logs showing a Event object left on the stack after the error is raised">
  <figcaption class="text-center">Logs showing a Event object left on the stack after the error is raised</figcaption>
</figure>

[bugsnag_L45]: https://github.com/bugsnag/bugsnag-ruby/blob/v6.26.1/lib/bugsnag/integrations/railtie.rb#L45

## 💡Understand the cause {#understand-the-cause}

There were two related problems, right next to each other.
The first was to do with `ActiveSupport::Subscriber`'s notion of `#children` and how it was tracking them.
[John Hawthorn][] explains it, and had [already fixed it in Rails 7.1][rails_pr_43390] (we were on 6.1 at the time).

This alone wasn't the cause our memory leak.
But in combination with a change to the Bugsnag Gem to [start cleaning URLs Rails breadcrumbs][bugsnag_pr_806], it manifested in a leak.
The Bugsnag change relied on `URI` to parse and then redact certain sensitive query strings.
The `URI` is rightfully pretty strict about what it considers a valid <abbr title="Uniform Resource Identifier">URI</abbr>, raising an error when given an invalid URI.
Meaning the `ActiveSupport::Notifications.subscribe` block in the Bugsnag Gem could now `raise` an error while processing an `ActiveSupport::Notifications::Event`.
And raising that error meant that the `parent` `Event` [wasn't getting popped off the `Subscriber#event_stack`][rails_L145].
So the `parent` `Event` was sticking around (leaking memory), and it was still holding a reference to the child `Event` via its `#children` `Array` (leaking more memory).

John's change not only removed the `Event#children` concept, but completely removed the shared-`Array` used to track `Events`.
The source of both leaks, gone in a single PR. 👏

[John Hawthorn]: https://jhawthorn.com/
[bugsnag_pr_806]: https://github.com/bugsnag/bugsnag-ruby/pull/806
[rails_L145]: https://github.com/rails/rails/blob/517ff4b7c6c6aeaa588993475277d816b0ba038e/activesupport/lib/active_support/subscriber.rb#L145
[rails_pr_43390]: https://github.com/rails/rails/pull/43390

## 🛠️ Fix the problem {#fix-the-problem}

John's fix for newer Rails means this is no longer a problem… on newer Rails versions.
But we weren't there yet.
As luck would have it, just days prior [Bugsnag had fixed][bugsnag_pr_811] the `Bugsnag.cleaner.clean_url` method to not `raise` an error on invalid URIs.

Our fix, in the short term, was to upgrade the Bugsnag Gem to a version that included the "don't raise an error while trying to clean an invalid URI" change.
Longer term we're upgrading Rails versions.

<figure>
  <img src="/assets/images/posts/memory_leak/puts-debugging-after.png"
       alt="Upgraded dependency no longer raises, and now the stack is empty"
       title="Upgraded dependency no longer raises, and now the stack is empty">
  <figcaption class="text-center">Upgraded dependency no longer raises, and now the stack is empty</figcaption>
</figure>

[bugsnag_pr_811]: https://github.com/bugsnag/bugsnag-ruby/pull/811

## 🤡 It's me, hi, I'm the problem, it's me. {#its-me-hi-im-the-problem-its-me}

<cite>(with apologies to Taylor Swift) 🙏</cite>

Funny story.
It turns out that Bugsnag change was introduced to our code base right around when the first memory spikes occurred.
We'd upgraded from `v6.26.0` to `v6.26.1` to fix a deprecation warning for a different dependency.
We (by which I mean me, I, myself) audited this change too.
And dismissed it as not being related because… why the heck would redacting part of a URI, something the Bugsnag Gem already did in other places, cause such a bad and sudden memory leak!?!?
Well… *something-something-something* assumptions. 🤦
