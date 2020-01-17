---
layout: post
title: "Debugging Homebrew with Pry"
date: 2020-01-17 16:00:01 -0500
comments: true
tags: [ruby, homebrew]
---

Over the years I've written a few [Homebrew formulas](https://github.com/Homebrew/homebrew-core/blob/master/Formula/git-tracker.rb) and sent the occasional Pull Request to update a formula or two.
But I've never done any work _within_ [Homebrew](https://brew.sh).
I've never needed to debug how Homebrew itself worked.
Until now.

I assumed our typical Ruby debugging tools, like [Pry](http://pryrepl.org) and [Pry-Byebug](https://github.com/deivid-rodriguez/pry-byebug) would work.
Homebrew is just Ruby, after all.
Which is true.
But it's also a bit special, and we can't do the normal `require "pry-byebug"; binding.pry` tricks.

But we can, with a little poking around, still use those tools!

<!-- more -->

### A Homebrew command

In this case, I was exploring a built-in command to see if I could extend it.
The [`brew-postgres-upgrade-database`](https://github.com/Homebrew/homebrew-core/blob/cb5e4b3ee75ae522360ddda5b27e67be67d56287/cmd/brew-postgresql-upgrade-database.rb) command is a great little tool that ✨ automagically ✨ upgrades a PostgreSQL instance's data between major versions.

```bash
$ brew postgresql-upgrade-database
==> Upgrading postgresql data from 11 to 12...
==> Moving postgresql data from /usr/local/var/postgres to /usr/local/var/postgres.old...
==> Creating database...
==> Migrating and upgrading data...
==> Upgraded postgresql data from 11 to 12!
==> Your postgresql 11 data remains at /usr/local/var/postgres.old
```

Except it only worked with the default `PGDATA` location.
Which is sensible.
But for… reasons, I had my `PGDATA` directory in a different location.
I keep mine within my user directory at `~/.pg-data`.

### A first, failed attempt

The first step was figuring out where this command lived on my machine so I could hack at it, as a proof-of-concept.
I needed it to allow an optional data directory.

```bash
$ brew --prefix
/usr/local
```

Knowing Homebrew was installed in `/usr/local` I went there and poked around until I found what I was looking for in `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/cmd/brew-postgresql-upgrade-database.rb`.
Phew, that's a mouthful!

From there I opened the code.

```bash
$ vim /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/cmd/brew-postgresql-upgrade-database.rb
```

Right above the existing code (snipped below for brevity) I tried the normal trick.

```diff
 #:  * `postgresql-upgrade-database`:
 #:    Upgrades the database for the `postgresql` formula.
+ 
+require "pry-byebug"; binding.pry
 
 name = "postgresql"
 pg = Formula[name]
```

Then I tried to run the command, hoping to pop a Pry session.

```bash
$ brew postgresql-upgrade-database
Error: cannot load such file -- pry-byebug
/usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/2.6.3/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
/usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/2.6.3/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/cmd/brew-postgresql-upgrade-database.rb:6:in `<top (required)>'
/usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/2.6.3/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
/usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/2.6.3/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
/usr/local/Homebrew/Library/Homebrew/utils.rb:82:in `require?'
/usr/local/Homebrew/Library/Homebrew/brew.rb:108:in `<main>'
```

😔 Well, crap.

### A second, successful attempt!

On a lark, I [searched the Homebrew code base for references to Pry](https://github.com/Homebrew/brew/search?q=pry&unscoped_q=pry), hoping for an Issue or someone else talking about this.
And it turns out, Homebrew has [some Pry support built it](https://github.com/Homebrew/brew/pull/3851/files).

```diff
 #:  * `postgresql-upgrade-database`:
 #:    Upgrades the database for the `postgresql` formula.
+ 
+ Homebrew.install_gem_setup_path! "pry"
+ Homebrew.install_gem_setup_path! "pry-byebug", executable: "pry"
+ require "pry-byebug"; binding.pry
 
 name = "postgresql"
 pg = Formula[name]
```

Leveraging that prior work I was able to get a full Pry Byebug session to pop.

```bash
$ brew postgresql-upgrade-database

From: /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/cmd/brew-postgresql-upgrade-database.rb @ line 10 :

     5: #:                    files. Same as the pg_ctl -D flag.
     6: Homebrew.install_gem_setup_path! "pry"
     7: Homebrew.install_gem_setup_path! "pry-byebug", executable: "pry"
     8: require "pry-byebug"; binding.pry
     9:
 => 10: name = "postgresql"
    11: pg = Formula[name]
    12: bin = pg.bin
    13: var = pg.var
    14: version = pg.version
    15: data_dir = options[:data_dir] || var/"postgres"
```

🎉 Huzzah!

### And then, a Pull Request

From there it was a matter of stepping through, line by line, to understand how the command worked.
Then some _hackity-hacking_ to allow an optional data directory to be passed in. 
The net result was a [Pull Request](https://github.com/Homebrew/homebrew-core/pull/49135) allowing you to upgrade a PostgreSQL database with a custom data directory.

```bash
$ brew postgres-upgrade-database --D ~/.pg-data
==> Upgrading postgresql data from 11 to 12...
==> Moving postgresql data from ~/.pg-data to ~/.pg-data.old...
==> Creating database...
==> Migrating and upgrading data...
==> Upgraded postgresql data from 11 to 12!
==> Your postgresql 11 data remains at ~/.pg-data.old
```

🤞 Fingers crossed that PR is merged.

Either way we (or at least I) learned how to debug Homebrew itself.
_Neat_.
