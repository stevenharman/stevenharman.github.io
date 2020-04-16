---
layout: post
title: "Well-behaved Ruby Objects: Equality"
date: 2020-04-14 12:40:01 -0400
comments: true
tags: [ruby]
description: Implement `hash` and `eql?` for use as a `Hash` key, and then `alias` the `eql?` to `==` for the expected developer ergonomics.
---

A "well-behaved" object in Ruby needs to understand the following:

What makes two Ruby objects "equal"?<br>
And which version of "equal" (there are several in Ruby)?<br>
And what makes an object usable as a `Hash` key?<br>
And is that the same thing that makes them `Comparable`?

Because I can never seem to remember the specifics.
And because my searching the Interwebs seems to find related, but not specific help.
And because I've got this blurgh-thing… I might as well use it to help future-me (and maybe you?).

## Gimme the gist

Implement `hash` and `eql?` for use as a `Hash` key, and then `alias` the `eql?` to `==` for the expected developer ergonomics. 
Something like this:

```ruby
class Message
  attr_reader :body, :subject

  def initialize(subject:, body:)
    @subject = subject
    @body = body
  end

  def eql?(other)
    other.class == self.class &&
      other.body == body &&
      other.subject == subject
  end
  alias == eql?

  def hash
    body.hash ^ subject.hash
  end
end
```

What to know more about the specifics, or how to also make these objects `Comparable`?
Read on, friend…

<!-- more -->

## Surely someone else has covered this already

As it turns out, Russ Olsen covers this very topic, in depth and with great examples, in his book, [Eloquent Ruby][eloquent-ruby].
I should know that because I've read that book, and recommend it!
However, it always takes me several trips to The Googles to find a snippet which eventually points me back to that book.
At which point I have to dig out my copy.
All of this only after I've tried a dozen other search terms resulting in articles about [the difference between the many forms of equality in Ruby][diff-equals].

So yes, to various degrees, it's been covered before.
There is nothing new under the Sun.

Like I said, I have this here blurgh and I'm gonna use it. 🤷

## Every-day object equality

For every-day use, the form of object equality we most often use in Ruby is the `==`, or double-equals, form.
By default, the `==` operator is an alias of the `equal?` method, which it inherits from `Object`.
This form compares the object identity, effectively asking "are these the same object in memory?"

We don't want this form, so we need to override it to something more meaningful for our object.
Often two objects are `==`, or double-equals equal, if they are of the same class, and some set of their internal data is also `==`.
In the case above, that means both objects are a `Message`, and their `body` and `subject` are the same.

```ruby
def ==(other)
  other.class == self.class &&
    other.body == body &&
    other.subject == subject
end
```

Please go read Mr. Olsen's excellent coverage of the topic for more details and in-depth examples.

## Use as a `Hash` key

When used as a `Hash` key, `Hash` will call the object's `hash` method to get hash code (an `Integer` value) to use when storing and searching for the key.
It then uses the object's `eql?` method to determine if two hash keys are the same key.
From the [Ruby docs][object-hash-docs]:

> **`hash` → integer**
>
> Generates an `Integer` hash value for this object. This function must have the property that `a.eql?(b)` implies `a.hash == b.hash`.
> 
> The hash value is used along with `eql?` by the `Hash` class to determine if two objects reference the same hash key. Any hash value that exceeds the capacity of an `Integer` will be truncated before being used.
> 
> The hash value for an object may not be identical across invocations or implementations of Ruby. If you need a stable identifier across Ruby invocations and implementations you will need to generate one with a custom method.

By default, objects inherit their `hash` and `eql?` methods from `Object`, which compares the object identity, similar to `equal?`.
As you might guess, we don't want that for our object.

For `hash`, we can lean on the internals already having their own `hash` values, and combine them.
We can use the exclusive-or operator (`^`) to take two other hashes (numbers) and smash them together in a consistent manner.

```ruby
def hash
  topic.hash ^ body.hash
end
```

To determine if two objects are `eql?`, and hence, the same keys in a hash, we want to make sure they're of the same class, and some set of their internal data is also `==`.

```ruby
def eql?(other)
  other.class == self.class &&
    other.body == body &&
    other.subject == subject
end
```

Look familiar?
It's the same as the `==` from above!

In practice, I often implement `eql?` and `hash` since they are a bit of a duo - one requiring the other.
And then I alias `==` to be the same as `eql?`


```ruby
def eql?(other)
  other.class == self.class &&
    other.body == body &&
    other.subject == subject
end
alias == eql?
```

And that's it, these objects now know if they are everyday-equal, and can serve as `Hash` keys.
We've not covered `===` here; the links above provide explanations about _when_, _why_, and _how_ you might need to reach for that form of equality amongst Ruby objects.
Again, go check out Mr. Olsen's book; it's great!

## What about `Comparable`?

Being able to compare, and thus `sort` two Ruby objects is a bit tangential to the forms of equality we've discussed here.

Objects with a "natural ordering" will implement the `<=>` or _spaceship_ operator.
This operator is used to determine if one object is less than the other, greater than the other, or equal to the other.
Ruby's `Comparable` module will use that to implement several comparison operators.
And all of that can be use by other Ruby modules, like `Enmerable` to do things like sort a collection.

_Neat!_

Again, Mr. Olsen's book, or any number of other articles, including the [Ruby docs for `Comparable`][ruby-comparable], cover this in great detail.

[diff-equals]: https://medium.com/@khalidh64/difference-between-eql-equal-in-ruby-2ffa7f073532 "Difference Between ==, eql?, equal? in ruby"
[eloquent-ruby]: https://amzn.to/2RC7Ph3 "Eloquent Ruby, by Russ Olsen"
[object-hash-docs]: https://rubyapi.org/2.7/o/object#method-i-hash "Ruby Docs: Object#hash"
[ruby-comparable]: https://rubyapi.org/2.7/o/comparable "Ruby Docs: Comparable"
