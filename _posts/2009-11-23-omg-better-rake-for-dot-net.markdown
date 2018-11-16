---
layout: post
title: "OMG, Better Rake (for .net)!"
date: 2009-11-23 13:58
comments: false
categories:
---
If you ask me, when it comes tools for writing automated build scripts
nothing packs more bang for the buck than [Rake][1]. Until recently,
using Rake to build .net solutions required a magic concoction of hacked
together scripts which rarely exhibited Ruby's appreciation for beauty
nor Rake's spirit of simplicity.

Luckily our buddy <a rel="met friend"
href="http://www.lostechies.com/blogs/derickbailey/" title="Derick
Bailey's blog">Derick Bailey</a> decided it was time to bite the bullet
and start building some *[real Rake tasks][2]* that were special suited
for building .net code. The result is [Albacore][3].

<!-- more -->

### Using Rake for .net <acronym title="In Real Life">IRL</acronym>

I've been using Rake to [be lazy][4] for a while. And we, the
[VersionOne][5] dudes & dudettes, have been using it to help automate
our <acronym title="Continuous Integration">CI</acronym> builds for over
a year now. And just last week we started ditching much of our
hacky-Rake-script inventory in favor of more concise, tested, and
readable Rake tasks via Albacore.

During the migration I've run into a few small hitches here and there,
but nothing that I couldn't track down, write a test for, and fix
within a couple of [tomatoes][6]. In one case I discovered an issue,
called Derick to confirm, suggested a fix, and had a new Albacore Gem
published within a couple of hours. *Hawt!*

Albacore already has a decent number of tasks baked in, and the list is
growing all the time!

*   [AssemblyInfoTask][7] - Generate an AssemblyInfo.cs file.
    Currently only supports C#
*   [ExpandTemplatesTask][8] - expand template files with #{setting}
    markers, using YAML configuration files as the data
*   [NCoverConsoleTask][9] - Run code coverage analysis through NCover's `NCover.Console`
*   [NCoverReportTask][10] - Check code coverage and get detailed
    reports through NCover's `NCover.Reporting`
*   NUnitTask - run NUnit test suites
*   [MSBuildTask][11] - Build a Visual Studio solution (`.sln`) or
    MSBuild file
*   [Rename Task][12] - Rename a file
*   [SftpTask][13] - Upload a file to a remote server via secure FTP
    connection
*   [SQLCmdTask][14] - Run scripts and other commands through SQL
    Server's `sqlcmd.exe`
*   [SshTask][15] - Run a command on a remote system via a secure
    shell connection
*   [ZipTask][16] - Package your build artifacts into a .zip for
    easier distribution source data

### Contribute!

As we move more and more of our custom stuff over I'll continue to add
features to Albacore, enhancing the great work the core team is doing.
In fact, I'm already planning a [NAnt task][17] to help those folks in
the process of migrating from an existing NAnt-based build script to
Rake. Look for it soon!

### Resources

*   [Albacore on GitHub][18]
*   [Albacore Wiki][19]
*   [OMG Rake!][20] - The original post which first inspired many to
    use Rake with .net

 [1]: http://rake.rubyforge.org/ "Rake - Ruby Make"
 [2]: http://www.lostechies.com/blogs/derickbailey/archive/2009/09/17/how-a-net-developer-hacked-out-a-rake-task.aspx "How A .NET Developer Hacked Out a Rake Task"
 [3]: http://github.com/derickbailey/Albacore "Albacore: A Suite Of Rake Build Tasks For .NET Solutions"
 [4]: http://stevenharman.net/blog/archive/2009/05/29/being-lazy-with-rake.aspx "Being Lazy with Rake"
 [5]: http://versionone.com/ "VersionOne: Simplifying Software Delivery"
 [6]: http://www.infoq.com/news/2009/09/Pomodoro "Pomodoro - An Agile Approach to Time Management"
 [7]: http://wiki.github.com/derickbailey/Albacore/assemblyinfotask
 [8]: http://wiki.github.com/derickbailey/Albacore/expandtemplatestask
 [9]: http://wiki.github.com/derickbailey/Albacore/ncoverconsoletask
 [10]: http://wiki.github.com/derickbailey/Albacore/ncoverreporttask
 [11]: http://wiki.github.com/derickbailey/Albacore/msbuildtask
 [12]: http://wiki.github.com/derickbailey/Albacore/renametask
 [13]: http://wiki.github.com/derickbailey/Albacore/sftptask
 [14]: http://wiki.github.com/derickbailey/Albacore/sqlcmdtask
 [15]: http://wiki.github.com/derickbailey/Albacore/sshtask
 [16]: http://wiki.github.com/derickbailey/Albacore/ziptask
 [17]: http://github.com/derickbailey/Albacore/issues/#issue/27 "Albacore::NAntTask - for migrating to Rake"
 [18]: http://github.com/derickbailey/Albacore "Albacore: A Suite of Rake Build Tasks For .Net Solutions"
 [19]: http://wiki.github.com/derickbailey/Albacore "Albacore Wiki"
 [20]: http://codebetter.com/blogs/david_laribee/archive/2008/08/25/omg-rake.aspx "OMG Rake!"
