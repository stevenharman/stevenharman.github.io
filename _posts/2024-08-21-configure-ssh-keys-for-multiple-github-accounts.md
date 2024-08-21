---
layout: post
title: "Configuring SSH Keys for Multiple GitHub Accounts"
date: 2024-08-21
comments: true
tags: [git, github, ssh]
image: /assets/images/posts/futurama-403-blue-key.webp
description: "Managing different SSH Keys for different Hosts is well-understood. But different keys for the same host (e.g., github.com), based on which Organization we're working with… that's less so."
---

Managing different <abbr title="Secure Shell">SSH</abbr> Keys for different Hosts is well-understood.
Different keys for the same host (e.g., `github.com`), based on which GitHub Organization we're working with, that's… a bit trickier.
But, we have the technology!

We're going to configure our SSH and Git clients to seamlessly switch SSH keys between our personal GitHub Account and our GitHub Enterprise Cloud, <abbr title="Enterprise Managed User">EMU</abbr> Account.

{% include figure.html src="futurama-403-blue-key.webp"
                       alt="Futurama character Fry and a soldier standing outside a 'War Room' door. A screen on the door has the words 'YOU NEED A BLUE KEY'. The soldier is holding up a comically large blue key."
                       caption="You need the right key." %}

<!-- more -->
## 🔐 Typical Multi-Key Setup

Most guides and information we'll find on the Internet about managing multiple SSH keys focus on multiple `Host` values.
As in, use key `A` for `example.com` and key `B` for `stevenharman.net`.
A simplified `~/.ssh/config` for that might look something like:

```
Host example.com
  IdentityFile ~/.ssh/id_a_ed25519

Host stevenharman.net
  IdentityFile ~/.ssh/id_b_ed25519
```

## 🤹 Different `HostName`s, Different Keys

But what happens when the `HostName`, i.e., the real host name that will be used for the SSH connection, for both is the same?
For example, when we have a personal GitHub Account (for Open Source work, some private repositories, and maybe even some repos belonging to other GitHub Organizations), and second [GitHub Enterprise Cloud][ghec] Account provisioned by our employer.

For brevity, let's say our personal GitHub Account is `fry`.
And our work for _Planet Express, Inc._, has a GitHub Enterprise Cloud instance, and provisioned us a `fry_plnx` [Enterprise Managed Users][emu] Account.

In both cases, the SSH `HostName` is `github.com`.
We might try a configuration like this:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_fry_ed25519

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_fry_plnx_ed25519
```

This configuration will work for `git` talking to GitHub for the `fry` Account, because it is matched first.
But when working with a repository for the `fry_plnx` Account, the same first `Host github.com` entry matches, and uses the `id_fry_ed25519` SSH Key.
But the `fry_plnx` Account doesn't know about that key, and so… we see errors.

```console
$ git clone git@github.com:planet-express/delivery_service.git
  ERROR: Repository not found.
  fatal: Could not read from remote repository.
  
  Please make sure you have the correct access rights
  and the repository exists.
```

[ghec]: https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud "About GitHub Enterprse Cloud"
[emu]: https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users "About Enterprise managed Users"

## 🤡 Use Different `Host` values

At this point, most articles on the Internet will tell us to [override the SSH command used][gh_multi] to connect, or (IMO, slightly better) [use different `Host` values][1p_multi] for each.
The later would look something like this:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_fry_ed25519

Host github-plnx
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_fry_plnx_ed25519
```

A big downside to this technique is that any time we're cloning a new repository for our `fry_plnx` Account, we'll need to adjust the <abbr title="Uniform Resource Locator">URL</abbr> used.
Meaning we can no longer copy/pasta the command from the GitHub <abbr title="User Interface">UI</abbr>.
Nor rely on muscle memory we've built up over years of using our personal Account.

```console
# Instead of the actual URL
$ git clone git@github.com:planet-express/delivery_service.git

# Substitue in our custom Host value for the `github.com` part
$ git clone git@github-plnx:planet-express/delivery_service.git
```

And this will work!
But we have to remember to manually substitute those values for each repository we clone.
It's toil.
And I detest toil.

[1p_multi]: https://developer.1password.com/docs/ssh/agent/advanced#use-multiple-github-accounts "SSH Agent: Use multiple GitHub accounts"
[gh_multi]: https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-personal-account/managing-multiple-accounts#contributing-to-multiple-accounts-using-ssh-and-git_ssh_command "Contributing to multiple accounts using SSH and GIT_SSH_COMMAND"

## 🤖 Automate Substituting the `Host`

I happened [across an article][kenmuse] using [Git's `url.<base>.insteadOf` variable][insteadof] to swap URLs.
We can use that to teach Git about our `Host github-plnx`, and have it auto-magically swap `github.com` back in when talking over SSH to GitHub.
We'll add this to our `~/.gitconfig`:

```
# See custom `Host github-plnx` in ~/.ssh/config
[url "github-plnx:planet-express"]
  insteadOf = git@github.com:planet-express
```

By including the `planet-express` GitHub Organization name in the match for the `url` variable, Git will alter the URL `git@github.com:planet-express/delivery_service.git` by matching `github.com:planet-express` and swapping in `github-plnx:planet-express`.
Then the SSH config matches the host `github-plnx`, and using that matched config, replaces `github-plnx` with the actual user (`git`) and host name (`github.com`).
At which point we're back to the original URL.
🎉 NEAT!

With that in place, Git commands for repositories under `github.com/planet-express` will use the `id_fry_plnx_ed25519` SSH key.
Any other repositories will fail to match the `github-plnx` host, thus using the regular `Host github.com` SSH Config, and the `id_fry_ed25519` key.

_NOTE: We'd need to add a similar entry for other Organizations in the GitHub Enterprise Cloud instance that we need to work with._

[insteadof]: https://www.git-scm.com/docs/git-config#Documentation/git-config.txt-urlltbasegtinsteadOf
[kenmuse]: https://www.kenmuse.com/blog/ssh-and-multiple-git-credentials/#git "SSH and Multiple Git Credentials"

### 🧹 Keeping `.gitconfig` Clean

If you've [got your dotfiles in Git][dotfiles], you might want to keep employer and/or machine-specific configuration out of your `~/.gitconfig`.
For example, our work machine needs to know about the custom `url "github-plnx:planet-express"` Git config, but our personal MacBook likely doesn't!
Or maybe we don't, or cannot, expose every GitHub Enterprise Cloud Organizations we're a part of.

Luckily, we can leverage [Git's ability to `include` other config files][git_includes] to keep such customizations out of Git.
We can adjust our `~/.gitconfig`, which is in our dotfiles Git repo, to include another config file which is _NOT_ in Git.
Add the following the bottom of our `~/.gitconfig`:

```
[include]
  path = ~/.gitconfig_custom
```

If this file isn't present, Git just ignores it.
As is the case on my personal MacBook.
😅 Phew!

When it is present, Git will load the file, allowing us to override or add additional configuration.
Now we move our `[url …]` configuration from `~/.gitconfig` into the new `~/.gitconfig_custom` file:

```
[url "github-plnx:planet-express"]
  insteadOf = git@github.com:planet-express
```

And _voila!_
We are switching SSH keys automatically, and our `~/.gitconfig` is clean of any knowledge about the specific GitHub Organizations, etc…

[dotfiles]: https://github.com/stevenharman/config?tab=readme-ov-file#my-config-files "Steven Harman's dotfiles"
[git_includes]: https://www.git-scm.com/docs/git-config#_includes "Git Config: include and includeIf"
