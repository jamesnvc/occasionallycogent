---
layout: post
title: "Exploring Io"
---

I first heard about the [Io programming language][iolang] quite a while ago -- I believe I saw it in [Steve Yegge's][steveyegge][^yegge] sidebar when I was still in high school and just starting to get interested in different programming languages.
Recently, it came up again on [Lobsters](https://lobste.rs) and I decided to have another look at it.

The first programming language I really got into when I first started learning to code was Scheme, so I really appreciate the minimalist "pure" feel of Io.
The way that everything is just sending messages to objects, with "syntax" like assignment operators just being macros for messages (e.g. `foo = bar` is sugar for `updateSlot("foo", bar)`) really appeals to me.
I also have a soft spot for the RPN-like notation that Io code tends to have -- the first open-source project I contributed to was [the Factor programming language][factor], so it's a nice dose of nostalgia for me.

To try make something real in Io, I decided to write a little command-line client for [Pinboard][pinboard].
Pinboard has a very simple API, so it seemed like a good target for making a little toy program.

Confusing things I ran in to:

  - `URL with("https:/...") fetch` didn't work -- default URL thing apparently doesn't support SSL.
    The fix was easy enough though, just used the `HCConnection` class & friends, which are mentioned in the reference.
  - I got a little thrown off by newlines being statement separators -- it makes sense, but I think I was thinking of it as too much of a concatenative language.
  - I like that I can see the slots objects have, but I'd love if I could also see docs for methods in-repl too (like Clojure's ability to look up vars' docstrings)
  - Trying to figure out how to parse CDATA from XML -- there is apparently an XmlReader module, but no documentation, I can't figure out how to make SGML do something sensible with CDATA content


  [iolang]: http://iolanguage.org/
  [steveyegge]: http://steve-yegge.blogspot.ca/
  [factor]: http://factorcode.org
  [pinboard]: https://pinboard.in

  [^yegge]: I owe a huge debt of gratitude to Steve Yegge: his [original blog](https://sites.google.com/site/steveyegge2/) was a huge influence on me when I first started programming and has really shaped the sort of developer I am today.
