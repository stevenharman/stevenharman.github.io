---
layout: post
title: "Caps Lock is Dumb; Make it Useful"
date: 2011-01-20 12:00
comments: true
categories:
---
I've long thought that `Caps Lock` was quite dumb. Yes, I'm sure there
is some archaic reason it exists, but the truth is I don't care. I don't
find it useful and am annoyed that it's taking up valuable room on my <a
title="The Home Row and Touch Typing"
href="http://en.wikipedia.org/wiki/Touch_typing" rel="external">Home
Row</a>. The more I use Vim the more angry I get at the `Caps Lock` key.

### Making Caps Lock Useful, on The Mac

I long ago remapped `Caps Lock` to `Esc` on my Mac - which worked
great for <a title="the missing editor" href="http://macromates.com/"
rel="external">TextMate</a>. However, these days I spend the majority of
my time in <a title="Vim for the Mac"
href="http://code.google.com/p/macvim/" rel="external">Vim</a> or <a
title="Zeee Shell" href="http://www.zsh.org/" rel="external">Zsh</a> (in
Vim mode) where I'd much prefer to have `Ctrl` on my Home Row.
Remapping `Caps Lock` to `Ctrl` is trivial on OS X; it's baked in
via *System Preferences > Keyboard Preferences > Modifier Keys*.

<!-- more -->

### Making Caps Lock Useful, on The Windows

To my knowledge, there is nothing baked into the <acronym
title="Operating System">OS</acronym> that makes this easy, but there
are a handful of utilities that will let you remap most keys. I opted
for the Lo-Fi route - hacking the registry to remap `Caps Lock` to
`Ctrl`.

```
REGEDIT4
 [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
 "Scancode Map"=hex:00,00,00,00,00,00,00,00,02,00,00,00,1d,00,3a,00,00,00,00,00
```

To use it, just download/save the raw `.reg` file to your Windows box,
and the run (double-click, whatever) it.

And there you have it. Good bye, annoying `Caps Lock`. Hello, useful
stuff!
