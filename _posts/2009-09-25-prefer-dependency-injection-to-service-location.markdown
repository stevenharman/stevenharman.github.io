---
layout: post
title: "Prefer Dependency Injection to Service Location"
date: 2009-09-25 14:17
comments: true
tags: [design, testing]
---
There is currently a thread running over in the StructureMap Users mailing list asking [if we really need constructor injection][1] when using an Inversion of Control container. Before any one rips off on a rant let me say that I worked with <a title="JonKruger.com" href="http://www.jonkruger.com/" rel="friend met co-worker">Jon</a> in my former life and I'm fairly certain he's merely conducting a thought experiment, trying to sure up his own beliefs. A worthwhile exercise, if you ask me.

At any rate, I have a few points I wanted to throw out there; most of them basic and mere reiterations of the words of others, but I'm gong to do it anyhow!

<!-- more -->

### The question at hand

I would encourage you to go read the full thread (it's a quick read... 4 minutes, tops!), but knowing many of you are lazy like me, I'll reprint Jon's original question here.

*Again, please go read the full thread so you have the full context.*

> Whenever I tell people about StructureMap (or using <acronym title="Dependency Injection">DI</acronym> in general), I mention that two of the benefits are that (a) StructureMap will create objects and all their dependencies for you and (b) it enables you to fake out the dependencies in a test.
> 
> Why do we need constructor injection to do this?  I can call `ObjectFactory.GetInstance()` anytime I want and it will work.  And I could leave SM configured for my tests and call `ObjectFactory.Inject()` to stub things out.
> 
> So theoretically, I wouldn't even need constructor injection, right?
>
> <cite>Jon Kruger - [StructureMap ML](http://groups.google.com/group/structuremap-users/browse_thread/thread/2ee1a7eab03d7f2a)</cite>

### Let's get the jargon down

To be clear, Jon proposing using [Service Location][2] rather than [Dependency Injection][3].

While Service location is better than [poor-man's DI][4], using it as suggested above is still introducing a high degree of coupling as all of these classes now have an opaque and highly concrete dependency on the container. This is effectively creating a new form of `Global`. *Eww!*

The key to using Service Location within **new code** is to keep it tucked away in the deepest, darkest corners of your infrastructure. For example, if you're building something on the asp.net mvc stack, you might use Service Location within a custom `IControllerFactory` to create each of your controllers.

If you're dealing with **legacy code**, full of concrete dependencies, you might use Service Location as technique for teasing things apart with a goal of decreased hard coupling. In the end this may result in wholesale replacement of some modules.

When it comes to Dependency Injection and dependencies in general, I agree with <a href="http://blog.scottbellware.com/" rel="friend met">Scott Bellware's</a> point of view; make your [dependencies explicit & transparent][5] by requiring them in the constructor. My gut reaction is also to avoid translucent (setter-injected) dependencies as they make it harder to tell what dependencies an object will need to do its job - the shape of the object isn't as clear as with explicit constructor dependencies.

### Feeling the friction

I tend to be lazy and prefer to feel friction of poor design early so I can change direction quickly. For example, when a constructor gets too large it's a signal to stop and consider [Single Responsibility Principle, Separation of Concerns, etc][6]. In a similar vein, I don't usually advocate use of an [auto-mocking container][7]. Or at least not for folks who've not yet acquired a strong nose for design and simplicity; the friction helps keep you on the rails.

Later in the tread Jon mentions some friction he's been feeling when setting up the [context][8] of his tests (or specs). Namely he's having to set up and inject a lot of concrete objects for interaction within his unit tests. To me this is an indication that those tests may actually be integration tests. After all, they are flexing the integration of a several modules in concert, right?

I say, call them what they are, fire up the fully configured container, and move on.

I prefer to make the implicit explicit, to be able easily see the shape of an object, and in getting forced feedback when my design starts to slip off the rails.

 [1]: http://groups.google.com/group/structuremap-users/browse_thread/thread/2ee1a7eab03d7f2a
 [2]: http://martinfowler.com/articles/injection.html#UsingAServiceLocator "Using a Service Locator"
 [3]: http://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection"
 [4]: http://www.lostechies.com/blogs/jimmy_bogard/archive/2009/07/03/how-not-to-do-dependency-injection-in-nerddinner.aspx
 [5]: http://codebetter.com/blogs/scott.bellware/archive/2007/06/28/164867.aspx "Dependency Patterns: Optional Dependencies and Primal Dependencies"
 [6]: http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod "Uncle Bob's Principles of Object Oriented Design"
 [7]: http://www.lostechies.com/blogs/joshuaflanagan/archive/2009/02/03/auto-mocking-explained.aspx "Auto-mocking Explained"
 [8]: http://stevenharman.net/blog/archive/2009/05/27/toward-a-better-use-of-context-specification.aspx "Toward a Better Use of Context/Specification"
