---
layout: post
title: "Toward a Better Use of Context/Specification"
date: 2009-05-27 06:42
comments: false
tags: [design]
---

If you've hand-rolled your own **Context/Specification** apparatus to
support your ~~test~~ spec-first lifestyle, you've likely got a base
class that looks something like the following:

```csharp
public abstract class concerns
{
  [SetUp]
  public virtual void setup_context()
  {
    context();
  }

  protected virtual void context() {}

  protected virtual void decontext() {}

  [TearDown]
  public virtual void cleanup_context()
  {
    decontext();
  }
}
```

The above is co-opting an existing unit testing tool into something more
language-oriented and behavior focused. In this case we’ve built upon
[MbUnit][], adding a couple of hook methods that are responsible for

1.  setting up the context before an individual specification -
    `context`
2.  optionally doing any necessary teardown after each specification –
    `decontext`

### An example

Using this base class, we’ll end up with specs that might looks
something like

```csharp
using Skynet.Core

public class when_initializing_core_module : concerns
{
  SkynetCoreModule _core;

  public void context()
  {
    //we'll stub it...you know...just in case
    var skynetController = stub<ISkynetMasterController>();
    _core = new SkynetCoreModule(skynetController);
    _core.Initialize();
  }

  [Specification]
  public void it_should_not_become_self_aware()
  {
    _core.should_not_have_received_the_call(x => x.InitializeAutonomousExecutionMode());
  }

  [Specification]
  public void it_should_default_to_human_friendly_mode()
  {
    _core.AssessHumans().should_equal(RelationshipTypes.Friendly);
  }

  // more specifications under this same context
  // ...
}
```

Here we’ve set up a common context that holds true for each of
specifications that follow it. This is also a common pattern used in
*classic unit testing* and in fixture-per-class style Test-driven
Development. In fact, the only real between the above and what I’d have
done in fixture-per-class style
<abbr title="Test-driven Development">TDD</abbr> is
the\_use\_of\_underscores, intention revealing names, and the context
hook method.

### Is that really any different?

These modest cosmetics are not what differentiate Context/Specification from
other styles of test-first development. For me, the real difference is the
realization that there are often many contexts under which a particular
behavior my be exercised, each producing an observable and possibly different
set of results.

With Context/Specification we’ll often have **many fixtures per
class/feature/functional area** of the code base. Doing this allows us to keep
the context as simple as possible and focused on the behavior being specified.
I’ve found that I tend to have a single file-per-class/functional area, with
any number of contexts (fixtures) in each file.

Another big distinction is that specifications should be side effect free. A
specification is an observation about the interactions that occurred or the
state of the system after some behavior has been exercised.

### Make it explicit!

We want small, focused contexts, yes? And we want side effect free
specifications too, yes? So why not leverage our tools to help guide us
in that direction? **YES!**

Consider the following tweak to the `concerns` base class

```csharp
public abstract class concerns
{
  [FixtureSetUp]
  public virtual void setup_context()
  {
    context();
  }

  protected virtual void context() {}

  protected virtual void decontext() {}

  [FixtureTearDown]
  public virtual void cleanup_context()
  {
    decontext();
  }
}
```

Such a base class will only set up each context once, no matter how many
specifications are made against the context. This does a couple of
things for us

-   requires side effect free specifications
-   guides us toward smaller, more focused contexts
-   might actually make our specs run faster!

As for the *running faster* bit, that is not guaranteed as it really
depends on how you were writing your specs before making this change.

### Some things to watch for

If, however, you were following more of a fixture-per-class style, you
might find a drastic reduction in how long it takes your spec suite to
run. The corollary is, of course, that you likely don’t have small
contexts. That is trouble and is often an indicator that the one, large
context is itching to be split out into two or more discrete contexts.

Upon switching your base class over to this more rigid
Context/Specification pattern, you might also find that you have some –
or many – broken specs. This is an indicator that those broken specs
were not side effect free. Well, actually its suggesting that some of
the sibling specs weren’t side effect free and they are now causing
other specs to break.

### Notes:

The portions of this article relating to changing from a standard
context set up to a once-per-fixture style apply to *most* of the
hand-rolled Context/Specification base classes I’ve seen in the wild.

If, however, you are using a tool like [MSpec][], then you’re in good
shape as [Aaron][] applied this same philosophy out of the gate. And if
you’re not using MSpec, I’d encourage you to take a look at it for
inspiration, if nothing else.

  [MbUnit]: http://mbunit.com/ "generative unit test framework"
  [MSpec]: http://github.com/machine/machine.specifications
  [Aaron]: http://aaron-jensen.com
