---
layout: post
title: "Trying Kakoune"
date: 2016-12-18 14:58:24
---

[Kakoune][kakoune] is an interesting new (to me, at least) text editor that aims at improving on vim's efficiency of text editing.

The way it works is somewhat similar to vim, with the big different being the switching of the "verb" and "object".
For example, deleting the next word in vim would be `dw`; the verb `d` is delete and after pressing that, vim will be waiting for the "object" on which to act -- in this case `w`, the next word.
In Kakoune, the order is reversed, so you'd first press `w` to select the next word, then `d` to delete the selection.
It seems to me like the main advantage is that this makes it easier to tell what you're operating on, but I'm not convinced it's that superior once one has internalized the way vim movement & selection works.
It's kind of like Kakoune's normal mode is vim's visual mode.
Not sure how I feel about that.

Kakoune's biggest feature that vim lacks is multiple cursors (à la Sublime Text).
The way it works seems pretty solid to me (I have never been anything but perplexed by Sublime) -- you make your selection then "split" the selection by provided a regex, resulting in each split having it's own cursor.
You can now move all the cursors in sync and edit from all of them.
There some built-in things that take advantage to this, like `&` to align all cursors by adding spaces, allowing you to do things like vim's [Tabular][tabs] extension by default.

The multi-selection is really neat and powerful, but I don't do multi-line editing particularly often -- and when I do, vim's `global` is usually enough for me (e.g. I visual select a region, then I can do `:'<,'>g/^/norm <stuff>` -- definitely more keystrokes, but not something I do very often).

One thing that immediately bothered me about Kakoune is it's lack of ability to do splits (i.e. view multiple buffers in a window simultaneously).
Instead (to simplify implementation, I think?) you are encourage to use your window manager or terminal multiplexer to create multiple windows, then have multiple Kakoune clients connect to the same session.
I really like being able to create splits in vim willy-nilly, to quickly look at another file, to get a temporary buffer, or whatever, so this more heavyweight approach to window manager isn't my cup of tea.
While I appreciate that it's kind of redundant when using tmux or a tiling window manager, Kakoune's approach makes it just un-ergonomic enough to discourage me from quickly opening splits.
I suppose that could be fixed with some tmux aliases though...

While Kakoune seems really cool and does a lot of neat stuff out of the box, I feel like it just isn't a big enough jump over neovim for me to make the switch.
It kind of reminds me of how I felt about Opera back in the day -- better out of the box than Firefox, but I'd already invested the effort in getting Firefox there, so why bother?
The built-in autocomplete is really nice, the way selections & multiple cursors work is very slick, I'm not sure if it's *that* much better than vim -- plus, (neo)vim just has so much more plugins and such already built for it.
Kakoune might be better from a blank slate, but my neovim setup, even after trimming a lot of unused plugins, currently has 68 different packages added to it, which is pretty hard to compete with.
While there is an extension API of sorts, it seems to essentially be "shell out to external tools" -- conceptually simple, but perhaps not well-suited for building something complex and interactive like [fugitive][] or [vim-fireplace][].

At this point, I think that unless your editor has [tpope][] using it, it's going to be a tough sell.

I'll keep my eye on Kakoune -- at the very least, it might be handy for some future bulk text surgery -- but I don't think I'll be switching from neovim right now.

  [kakoune]: http://kakoune.org/
  [tabs]: https://github.com/godlygeek/tabular
  [fugitive]: https://github.com/tpope/vim-fugitive
  [vim-fireplace]: https://github.com/tpope/vim-fireplace
  [tpope]: https://github.com/tpope/
