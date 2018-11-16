---
layout: post
title: "In Search Of... Enumerable#transform for Ruby"
date: 2018-11-16 10:56:01 -0500
comments: true
categories:
---

**In Search Of…** a Ruby method that mixes the semantics of `Enumerable#map` and `Enumerable#inject` to transform values, while also having access to the prior iteration return value.
Sounds odd, I know.

Given the following "value object"

```
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
Right now (as of Ruby 2.5), I think that means doing one of the following:

* A) Use an initial value, and re-assign it in each iteration

    ```
    snap = Snapshot.new
    (1..10).map { |i| snap = snap.apply(i) }
    ```

* B) Use an awkward initial setup (pre-calculating the first iteration), and juggle the real value in the block

    ```
    (2..10).inject([Snapshot.new(1)]) { |snaps, i| snaps.push(snaps.last.apply(i)) }
    ```

Either of the options product the following:

```
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

I'd like something that looks like this, maybe?

```
(1..10).transform(Snapshot.new) { |prev_snap, i| prev_snap.apply(i) }
```

Is there such a thing?
If not, how might such an API look?
I'm not sure `#transform` is a great name, nor if a new method is required.
Perhaps a new optional argument to `#map` would be sufficient?

```
(1..10).map(Snapshot.new) { |i, prev_snap| prev_snap.apply(i) }
```

This would mean there'd need to be a default initial value, of course.
`nil`, perhaps?
Or maybe follow `#inject`'s lead and if no initial value is provided, use the first value of the collection?

> If you do not explicitly specify an initial value for memo, then the first element of collection is used as the initial value of memo.
> <cite>http://ruby-doc.org/core-2.5.3/Enumerable.html#method-i-inject</cite>
