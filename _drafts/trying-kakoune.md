---
layout: page
title: "Trying Kakoune"
---

[Kakoune][kakoune] is an interesting new (to me, at least) text editor that aims at improving on vim's efficiency of text editing.

The multi-selection is really neat and powerful, but I don't do multi-line editing particularly often -- and when I do, vim's `global` is usually enough for me (e.g. I visual select a region, then I can do `:'<,'>g/^/norm <stuff>` -- definitely more keystrokes, but not something I do very often).

It kind of seems like Kakoune's normal mode is vim's visual mode.
Not sure how I feel about that.

I really like being able to create splits in vim willy-nilly, to quickly look at another file, to get a temporary buffer, or whatever.
While I appreciate that it's kind of redundant when also using tmux, kakoune's approach makes it just heavyweight enough to discourage me from quickly opening splits, although I suppose that could be fixed with some tmux aliases...

While Kakoune seems really cool and does a lot of neat stuff out of the box, I feel like it just isn't a big enough jump over neovim for me to make the switch.
It kind of reminds me of how I felt about Opera back in the day -- better out of the box than Firefox, but I'd already invested the effort in getting Firefox there, so why bother?
The built-in autocomplete is really nice, the way selections & multiple cursors work is very slick, I'm not sure if it's *that* much better than vim -- plus, (neo)vim just has so much more supporting stuff already built for it.
Kakoune might be better from a blank slate, but my neovim setup, even after trimming a lot of unused plugins, currently has 68 different packages added to it, which is pretty hard to compete with.

I think I'll keep my eye on Kakoune -- at the very least, it might be handy for some future bulk text surgery.

  [kakoune]: http://kakoune.org/
