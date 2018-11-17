---
layout: post
title: "Git Pull with Automatic Rebase"
date: 2011-06-09 22:57
comments: true
categories:
---

To rebase, or not to rebase - for me its not really a question. I
generally prefer a clean, linear commit history. Why? Because merge bubbles make
history confusing, noisy, and can break `git bisect`.

![Y U NO REBASE!?!](/assets/images/posts/y-u-no-rebase-preview.jpg){:class="img-thumbnail img-thumbnail--right"}
Don't believe me? Check out the pretty log to the right. See all
those merge bubbles in there? *Eww!*

<h3>The Why?</h3>

The workflow that caused those merges was as follows:

1. `git pull` (to bring local up to date)
2. *hackity-hack*
3. `git commit`
4. `git pull`
5. `git push`

By default `git pull` will fetch any new commits from the remote, and
then merge any local changes in, resulting in the merge bubbles.

<!-- more -->

<h3>A better approach</h3>

I typically use the same workflow as above with one tweak.
Rather than `git pull` I use `git pull --rebase`.
The `--rebase` option will fetch the remote commits and rebase your commits on top of the new commits from the remote.
This is the "re-writing" of history folks often talk about.

<h3>Make it better, automatically!</h3>

You can tell git to use rebase, rather than merge, in one of two ways, depending on
your situation.

```
$ git config branch.autosetuprebase always # Force all new branches to automatically use rebase
```

You can add the `--global` switch to have all future branches, in all
repositories on this machine, behave this way.

```
$ git config branch.*branch-name*.rebase true # Force existing branches to use rebase.
```

<h3>Get more info</h3>

Be sure to check out the <a title="git-config Manual Page" href=
"http://www.kernel.org/pub/software/scm/git/docs/git-config.html" rel="external">git
man pages</a> for more info on what those options mean and when you may or may not want
to use them.

You might also want to check out my <a title="Git Workflows" href=
"https://github.com/stevenharman/git-workflows" rel="external">Git Workflows repository
on The GitHubs</a> where you can find a Keynote presentation (or PDF in the Downloads)
explaining `git rebase` vs. `git merge` Complete with
pictures!
