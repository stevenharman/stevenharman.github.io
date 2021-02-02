---
layout: post
title: "Verbose Shell Scripts for Future You"
date: 2021-02-01
comments: true
tags: [shell]
image: /assets/images/posts/curl-example.jpg
description: "When writing shell scripts prefer long options; make things easier for others who come later, including future you."
---

Command line tools and [shell builtins][shell_builtins] can offer both a short- and long-form way to specify options.
Sometimes these are also called sort and long flags, respectively.
The upshot is a user experience with two ways to specify the same option.
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

They're more terse.
Literally, they are less to type, less to read, and _smaller_.

These can be desirable traits when writing a one-off commands in your shell.
With less, to type you can execute the command sooner.
Seems Good™️.

A drawback of this terseness is a decrease in readability and increase in cognitive load.
That is to say, they are more difficult for a human (or at least _this_ human) to parse and internally translate into a meaningful "name."
Consider again the example I opened with:

```console
$ curl -HLsS http://stevenharman.net
# ⬆ is the same as ⬇
$ curl --head --location --silent --show-error http://stevenharman.net
```

To my eyes and brain, parsing and understanding that first line is difficult, at best.
Examples like the casing of `s` vs `S` referring to different, but related options, abound.
It's also difficult to _notice_ the difference in casing when scanning the code.
Especially days, weeks, or years later, when you've totally forgotten that there is a difference.

But hey, if I'm just ripping off a quick `cURL` to check if my site is up, who cares? 
Presumably _I_ know and remember what each of those `-HLsS` options are doing, so I saved myself a bunch of typing.
Which for me also means a bunch of typos. 🎉

In summary, short options less to write and read, at the expense of not being as obvious or self-describing.

## Long Options are Good, Too!

Well, they're longer, no doubt about that!

I find that makes them easier to read at a glance.
And their names help to tell _what they do_ - why they're being used, in the context.
Both properties are immensely valuable when used in long-lived scripts.
For example, when as a part of a shell script that my team uses for daily operations.
Or perhaps even worse, a script we use rarely, but which does some crucial task.

The glance-ability and memorability of the longer names also pays off when searching for a one-liner after the fact.
You might remember an option name and use it as a keyword to [reverse fuzzy search][fzf] (<kbd>^R</kbd>) your command history.
A trick I use ALL THE TIME!

![Shell reverse history fuzzy search with Zsh Navigation Tools](/assets/images/posts/curl-shell-history-example.gif){:class="img-thumbnail img-thumbnail--center"}

Above I search `curl data` and quickly find a prior `cURL` command that posted some JSON data.
Exactly what I needed (because I can never remember exactly how that works).
The same goes for the many CLIs and their innumerable options that I use on a daily basis.

A drawback to the verbosity is there's more to type!
Which I suppose leads to more typos and [non-zero exit codes][zero_exit_code].
Though, if we're optimizing for read-time, rather than write-time, maybe that isn't such a big deal. 🤷

## Which Style to Use, and When?

Whatever history might have to say about _why_ we have long options, I prefer to use them when they're available.
That goes for both one-off commands in the shell, and doubly so for scripts that will exist beyond their initial authoring.
Admittedly, sometimes I am a bit lazy, or overly rely on muscle memory, and fall back to short options. 🙈

If you don't already, I'd encourage you to try using long options, especially for scripts.
But like I said, neither is right nor wrong.
It Depends™️.
Consider the trade offs, and move forward.

[cli_complexity]: https://danluu.com/cli-complexity/ "The growth of command line options, 1979-Present"
[fzf]: https://github.com/junegunn/fzf#- "fzf: Command-line Fuzzy Finder"
[gnu]: http://en.wikipedia.org/wiki/GNU "GNU: GNU's Not Unix"
[shell_builtins]: https://en.wikipedia.org/wiki/Shell_builtin "Shell builtin"
[zero_exit_code]: https://en.wikipedia.org/wiki/Exit_status "Exit status"
