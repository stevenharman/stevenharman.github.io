---
layout: post
title: "Git Clean: Delete Already-Merged Branches"
date: 2013-10-01 15:20
comments: true
categories:
---
## TL;DR

To delete local branches which have alread been merged into `master`:

    $ git branch --merged master | grep -v "\* master" | xargs -n 1 git branch -d

You can omit the `master` branch argument to remove local branches which have
already been merged into the current `HEAD`:

    $ git branch --merged | grep -v "\*" | xargs -n 1 git branch -d

## Breaking it down

We start by getting a list of local branches which have already been merged
into the current branch (i.e. `HEAD`)

    $ git branch --merged

      add_new_user_gravatar_links
      assign_unique_key_to_uploads
    * master
      remember_the_last_activity_per_user
      update_kaminari_to_thread_safe_version

We then pipe that to `grep` to match on the `"\*"` character, inverting that
match via `-v` to get all merged branches sans the current one.

    $ git branch --merged | grep -v "\*"

      add_new_user_gravatar_links
      assign_unique_key_to_uploads
      remember_the_last_activity_per_user
      update_kaminari_to_thread_safe_version

Finally we pipe that list in to `xargs` so we can strip apart the input and
pass it on to a new command. We use `-n 1` to ensure at most one argument is
taken from the input to be passed to the invocation of the new command. The
resulting commands that `xargs` will invoke are effectively

    $ git branch -d add_new_user_gravatar_links
    $ git branch -d assign_unique_key_to_uploads
    $ git branch -d remember_the_last_activity_per_user
    $ git branch -d update_kaminari_to_thread_safe_version

Pulling it all back together, we have

    $ git branch --merged | grep -v "\*" | xargs -n 1 git branch -d

    Deleted branch add_new_user_gravatar_links
    Deleted branch assign_unique_key_to_uploads
    Deleted branch remember_the_last_activity_per_user
    Deleted branch update_kaminari_to_thread_safe_version

And there you have it. Go forth and clean up!
