---
layout: post
title: "A Rails 7 compatible bin/dev for heroku local"
date: 2022-09-01
comments: true
tags: [shell, rails]
image: /assets/images/posts/bin-dev-heroku-local.jpg
description: "Rails 7 introduced a new `bin/dev` wrapper to launch and manage your Rails server, CSS watcher, and JS bundler into a single process, managed by `foreman`. Here's a `heroku local` (or `node-foreman`) version of the same."
---


Rails 7 introduced a new `bin/dev` wrapper to launch and manage your Rails server, CSS watcher, and JS bundler into a single process, managed by [`foreman`][foreman].
This is quite handy for running everything with a single command.
But what if you're deploying to Heroku and using the Heroku CLI's [`heroku local`][heroku-local] to run things locally?
Or if you're a fan of one of the other tools that manage processes based on your `Procfile`?

I've got you covered!

## `bin/dev` for `heroku local`

The biggest, and most obvious change is swapping out the executable.
Mostly this is to do with `heroku local` using the `:<subcommand>` style CLI interface.
Let's compare `foreman`, `heroku local`, and `bin/dev` doing similar things to see how their CLIs differ.


```console
# start the web and css processes
$ foreman start web,css
$ heroku local:start web,css
$ bin/dev web,css
```

It turns out `bin/dev` explicitly calls the `foreman start` command.
So a quick `heroku local` version of the same would be:

<!-- more -->

```sh
#!/usr/bin/env sh

if ! command -v heroku &> /dev/null; then
  printf "You must install the Heroku CLI on your system before setup can continue\n"
  printf "☁️  https://devcenter.heroku.com/articles/heroku-cli\n"
  exit 1
fi

exec heroku local:start "$@"
```

And this works… fine-ish.
I mean, it totally works.
But it's only using a small part of the underlying `heroku local` CLI.
And the same goes for the `foreman` version.
If we want to leverage the `run` or `version` commands, we're out of luck.

## A more robust `bin/dev`

If we want to leverage the other CLI commands, we're going to need to do some `bash`-ing to check and manipulate the passed arguments.
Let's start with the CLI we _want_ to have.

```console
$ bin/dev web,css         # (implicit) start the web and css processes
$ bin/dev start web,css   # (explicit) start the web and css processes
$ bin/dev run bin/rails c # start a rails console session
$ bin/dev version         # check the foreman version
```

For this we need to default to the `start` command, but allow overriding it.
And we'll need to pass other arguments on to `heroku local`.

```sh
#!/usr/bin/env bash

if ! command -v heroku &> /dev/null; then
  printf "You must install the Heroku CLI on your system before setup can continue\n"
  printf "☁️  https://devcenter.heroku.com/articles/heroku-cli\n"
  exit 1
fi

cmd='start'
heroku_cmds=('run' 'start' 'version')

# If we were given any positional args we might need to handle sub-commands
if [[ $# -gt 0 ]];then
  first_arg=${1}

  # Shift heroku local commands off the array so we can pass remaining arguments along
  if [[ " ${heroku_cmds[*]} " == *" ${first_arg} "* ]]; then
    cmd="${first_arg}"
    shift
  fi
fi

exec heroku "local:${cmd}" "$@"
```

🎉 And just like that, it works!
We've got a `bin/dev` that works with `heroku local` _and_ supports its other commands.
You can even `bin/dev --help` to see `heroku local`'s CLI help text.

I'll leave it as an exercise to the reader to adapt this for [`node-foreman`][node-foreman] or other tools.

[foreman]: https://github.com/ddollar/foreman "Manage Procfile-based applications"
[heroku-local]: https://devcenter.heroku.com/articles/heroku-local "Use the Heroku CLI to run Apps locally"
[node-foreman]: https://github.com/strongloop/node-foreman "Node Foreman is a Node.js version of the popular Foreman tool, with a few Node specific changes."
