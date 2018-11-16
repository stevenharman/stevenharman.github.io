---
layout: post
title: "An Io Language Vim Plugin"
date: 2011-02-11 16:27
comments: false
categories:
---

Who here doesn't enjoy a little color in their life? I know I do, especially when
used to highlight the syntax of a language - as anyone who's been around me while
downing a few pints can attest!

<h3>Learning, Io, and Vim</h3>

[![Io Syntax Highlighting in Vim](/assets/images/posts/io-syntax-thumb.png){:class="left"}](/images/posts/io-syntax.png)

In an attempt to feed our insatiable desire to learn, a
few of us at <a title="VersionOne: Simplifying Software Delivery" href=
"http://versionone.com" rel="external">VersionOne</a> are doing a book club on
<a title="Seven Languages in Seven Weeks: A Pragmatic Guide to Learning Programming Languages"
href=
"http://www.amazon.com/gp/product/193435659X?ie=UTF8&amp;tag=stevenharman-20&amp;linkCode=as2&amp;camp=1789&amp;creative=390957&amp;creativeASIN=193435659X"
rel="external">Seven Languages in Seven Weeks</a>. We're currently working on chapter
2: <a title="Io Language" href="http://iolanguage.com/" rel="external">Io</a>. My
current favorite editor is Vim. I wanted syntax highlighting for Io, in Vim.

I found a decent Vim script to get Io syntax highlighting, and then wrote a quick
`ftdetect` script to set Io-related files to use the Io syntax. The
resulting vim-io plugin is currently embedded <a title="vim-io: Io, for Vim!" href=
"https://github.com/stevenharman/config/tree/master/.vim/bundle/vim-io" rel=
"external">in my dotfiles on the GitHubs</a>, but if there's interest I can pull them
out into a standalone plugin.

Grab it, enjoy it, fork and improve it!
