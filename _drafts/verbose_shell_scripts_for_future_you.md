---
layout: post
title: "Verbose Shell Scripts for Future You"
date: 2020-12-19
comments: true
tags: [shell]
image: /assets/images/posts/curl-example.jpg
description: "When writing shell scripts prefer long flags; make things easier for others who come later, including future you."
---

Command line tools and [shell builtins][shell_builtins] can offer both a short- and long-form way to specify options.
Sometimes these are also called sort and long flags, respectively.
Either way, the user experience allows two ways to specify the same option.
A short and terse way.
And a longer, more verbose way.

```console
$ curl -HLsS http://stevenharman.net
# ⬆ is the same as ⬇
$ curl --head --location --silent --show-error http://stevenharman.net
```

Both ways are useful, neither right nor wrong, and each with their own trade offs to consider.

## Why Both?

![Meme: Girl shrugging and asking, "Why don't we have both?"](/assets/images/posts/why-dont-we-have-both.gif){:class="img-thumbnail img-thumbnail--right"}{:width="300"}
_Why are there two ways to say the same thing?_


It's a good question.
One I've asked too.
But I've not found a definitive answer.
Perhaps the truth is lost to history at this point?

From what I can tell, _originally_ (i.e., the early/original *nix flavored OSes) there were only single-character options.
At some point multi-character options were added, possibly via [GNU][gnu].

Along the way the sheer [number of options also grew, a lot][cli_complexity].
Is this a cause and effect relationship, a correlation, or pure happenstance?
I don't know.
But today we have A LOT of options, and often multiple ways to express them.

Let's talk about the trade offs and when to use each style.

<!-- more -->

## Short Options are Good.

Off the bat, they're more terse.
Literally, they are less to type, less to read, and _smaller_.

These can be desirable traits when writing a one-off commands in your shell.
You less to type, so you can execute your command sooner.
Seems Good™️.

A drawback of this terseness is a decrease in readability and increase in cognitive load.
They are both more difficult for a human (or at least _this_ human) to parse and translate into a "name" for the option.
Consider again the example I opened with:

```console
$ curl -HLsS http://stevenharman.net
# ⬆ is the same as ⬇
$ curl --head --location --silent --show-error http://stevenharman.net
```

To my eyes and brain, parsing and understanding that first line is difficult, at best.
But hey, if I'm just ripping off a quick `cURL` to check if my site is up, who cares? 🚀
Right?
_I_ presumably know and remember what each of those `-HLsS` options are doing, so I saved myself a bunch of typing.
Which for me also means a bunch of typos.

## Long Options are Good, Too!

Whatever history might have to say about _why_ we have long options, it's clear to me that they're easier to read and understand.
Especially after the fact.
For example, when used as part of a shell script that my team uses for daily operations.
Or perhaps even worse, a script we use rarely, but which does some crucial task.

I find myself reading my own quick, one-off commands ALL THE TIME by way of my shell's reverse fuzzy command history.

![Shell reverse history fuzzy search with Zsh Navigation Tools](/assets/images/posts/curl-shell-history-example.gif){:class="img-thumbnail img-thumbnail--center"}

By using long-form options when available, I can leverage those more readable and memorable names to search my command history.
Above I search `curl data` and quickly find a prior `cURL` command that posted some JSON data.
Exactly what I needed, because I can never remember exactly how that works!

It's a similar to story to so many CLIs I use on a day to day basis.

A drawback to that verbosity is the inverse of the big advantage of the short-form: there's more to type!
If we're optimizing for read-time, rather than write, maybe this isn't such a big deal. 🤷

## Which Style to Use, and When?

Personally, I prefer to use long options when they're available.
That goes for both one-off commands in the shell, and doubly so for scripts that will exist beyond me writing them.
Admittedly, sometimes I am a bit lazy, or overly rely on muscle memory, and fall back to short options for those one-offs. 🙈

I'd encourage you to also prefer long options, especially for scripts.
But like I said, neither is right nor wrong.
It Depends™️.
Consider the trade offs, and move forward.

[cli_complexity]: https://danluu.com/cli-complexity/ "The growth of command line options, 1979-Present"
[gnu]: http://en.wikipedia.org/wiki/GNU "GNU: GNU's Not Unix"
[shell_builtins]: https://en.wikipedia.org/wiki/Shell_builtin "Shell builtin"
