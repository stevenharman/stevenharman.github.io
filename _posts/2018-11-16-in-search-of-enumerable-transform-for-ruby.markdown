---
layout: post
title: "In Search Of... Enumerable#transform for Ruby"
date: 2018-11-16 10:56:01 -0500
comments: true
tags: [ruby]
---

_In Search Of…_ a Ruby method with the semantics of `Enumerable#map` and `Enumerable#inject`.
That is, to transform values while also having access to the prior iteration return value.
Sounds odd, I know.

Given the following "value object"

```ruby
class Snapshot
  def initialize(value = 0)
    @value = value
  end

  def apply(new_value)
    self.class.new(@value + new_value)
  end
end
```

I want to build an array of snapshots, based on some set of inputs.
Meaning, I _think_ I'd like something that looks like this

```ruby
(1..10).transform(Snapshot.new) { |prev_snap, i|
  prev_snap.apply(i)
}
```

Resulting in this

``` irb
#=> [#<Snapshot:0x00007fe550d12558 @value=1>,
#=>  #<Snapshot:0x00007fe550d12508 @value=3>,
#=>  #<Snapshot:0x00007fe550d124e0 @value=6>,
#=>  #<Snapshot:0x00007fe550d124b8 @value=10>,
#=>  #<Snapshot:0x00007fe550d12468 @value=15>,
#=>  #<Snapshot:0x00007fe550d12440 @value=21>,
#=>  #<Snapshot:0x00007fe550d12418 @value=28>,
#=>  #<Snapshot:0x00007fe550d123c8 @value=36>,
#=>  #<Snapshot:0x00007fe550d123a0 @value=45>,
#=>  #<Snapshot:0x00007fe550d12378 @value=55>]
```

Right now, as of Ruby 2.5, I don't know of a clean way of doing that.
_Clean_ being in the eye of the beholder, I suppose.
It can be done with `Enumerable#map` or `Enumerable#inject`, but it ain't pretty.

<!-- more -->

#### `Enumerable#map`

Here we set an initial value, and re-assign it in each iteration.

```ruby
snap = Snapshot.new
(1..10).map { |i| snap = snap.apply(i) }
```

Which works, but setting up the initial value only so we can re-assign it a new value each iteration feels odd.

#### `Enumerable#inject`

In this case we use an awkward initial setup (pre-calculating the first iteration), and juggle the real value in the block.

🎩 _Hat tip to [Matt Jones](https://twitter.com/al2o3cr) for showing me this one._

```ruby
(2..10).inject([Snapshot.new(1)]) { |snaps, i|
  snaps.push(snaps.last.apply(i))
}
```

On the up side, both options product the following, desired output!

```irb
#=> [#<Snapshot:0x00007fe550d12558 @value=1>,
#=>  #<Snapshot:0x00007fe550d12508 @value=3>,
#=>  #<Snapshot:0x00007fe550d124e0 @value=6>,
#=>  #<Snapshot:0x00007fe550d124b8 @value=10>,
#=>  #<Snapshot:0x00007fe550d12468 @value=15>,
#=>  #<Snapshot:0x00007fe550d12440 @value=21>,
#=>  #<Snapshot:0x00007fe550d12418 @value=28>,
#=>  #<Snapshot:0x00007fe550d123c8 @value=36>,
#=>  #<Snapshot:0x00007fe550d123a0 @value=45>,
#=>  #<Snapshot:0x00007fe550d12378 @value=55>]
```

#### `Enumerable#transform`, maybe? 🤔

So, how about that `Enumerable#transform` I mentioned?
Well, it's completely made up.
By me.
Just now.

```ruby
(1..10).transform(Snapshot.new) { |prev_snap, i|
  prev_snap.apply(i)
}
```

I'm not particularly sold on the name `#transform`, nor the other particulars.
Perhaps a new method isn't even needed?
An optional argument to `#map` might be sufficient?

```ruby
(1..10).map(Snapshot.new) { |i, prev_snap|
  prev_snap.apply(i)
}
```

This would mean there'd need to be a default initial value, of course.
`nil`, perhaps?
Or maybe follow `#inject`'s lead and if no initial value is provided, use the first value of the collection?

> If you do not explicitly specify an initial value for memo, then the first element of collection is used as the initial value of memo.
> <cite>[Ruby Docs `Enumerable#inject`](http://ruby-doc.org/core-2.5.3/Enumerable.html#method-i-inject)</cite>

#### Or… maybe not? 🤷

I'm not sure if this is needed.
Nor how useful it would be.

What I do know is I reached for something like this today, and to my surprise, it wasn't there.
I've become so accustomed to Ruby (sometimes with the aide of `ActiveSupport`) having just the ditty I need, that I'm surprised when it doesn't.

> The goal of Ruby is to make programmers happy. I started out to make a programming language that would make me happy, and as a side effect it’s made many, many programmers happy.
>
> I hope to see Ruby help every programmer in the world to be productive, and to enjoy programming, and to be happy. That is the primary purpose of Ruby language.
> <cite>Yukihiro "Matz" Matsumoto</cite>

What say you?
A potential addition for Ruby `vNext`?
Or am I expecting too much programmer happiness from this great language?
