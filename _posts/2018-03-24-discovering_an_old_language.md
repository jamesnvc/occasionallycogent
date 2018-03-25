---
layout: post
title: Discovering an Old Language
date: 2018-03-24
---

I've been playing around with learning Prolog for the past few weeks and have been really, really enjoying it.

I'd heard of Prolog before, but mostly just that it was a neat logic language, that wasn't very suited to any real work.
However, I was re-reading *Coders at Work* and was kind of curious that Joe Armstrong, the creator of Erlang, spoke about how much he liked Prolog, and how, while Erlang was built on top of it, just straight-up Prolog was a pretty solid language.
I've tried out Elixir recently and had been kind of annoyed by it [once][elixirwoes1] or [twice][elixirwoes2], so was thinking that maybe the next language I'd try learning would be Erlang or maybe even Prolog.

While Erlang was easy enough to find resources on learning though, I wasn't really sure where to start with Prolog, as all the tutorials I came across were for university courses and all about like, creating genealogy tables but not so much sending HTTP requests.

However, shortly after this, I came across a very interesting [Twitter thread from @AnnieTheObscure][prolog_facts], listing a bunch of interesting things about Prolog (specifically SWI-Prolog).
I was really curious about this, especially the part where she said:

> Prolog is taught terribly. Writing 'real' Prolog code is not a mind bending puzzle. What is taught in PL classes is crazy tricks.

After reading this thread, I started going through her [web app in Prolog tutorial][web_app_tut] as well as [some of her other ones][clpfd_tut] and really, really enjoyed the experience.
While there was a bit of mind-bending stuff in a few places, I found it mostly very straight-forward and delightfully declarative.

I've been mainly using Clojure for the past few years, so I thought I was used to pretty minimal syntax & declarative programming, but Prolog takes it to another level.

There are just so many things about it that I find so satisfying:
The way that most things aren't indented more than one level makes following the flow of the logic very straightforward;
the inclusion of DCGs is delightful -- it's like instead of having syntax for regular expressions, you have a syntax for parser generators, which is fantastic;
[CLP(FD)][clpfd] is a really cool way of describing constraints over numbers ("Constraint Logic Programming over Finite Domains") so, for example, you can express things like "X is between 0 and 100, Y is X + 5, X modulo 7 is 3" and see all possible solutions;
[pengines][], which I'm just learning about now, essentially let you easily expose a module over RPC;
the underlying syntax is surprisingly very regular, using M-exprs instead of Lispy S-exprs, so it's actually very easy to dynamically re-write code, as you would with Lisp macros;
and [SWI Prolog][swi-prolog] itself is a very solid distribution, with tons of built-in libraries.

So far, I've built a [Braid scheduling bot/extension][schedulebot] in Prolog, in the process making libraries (which, by the way, it was trivial to add to the community package repository) for [MessagePack][msgpack] and [UUID generation][uuid].
Prolog not only was very easy to work with, but it meets the rare bar of making me happy to be writing in.

I'm planning on continuing to make more things in Prolog (I'm particularly interested in rich web apps using pengines...) and I really encourage you to give it a try!

  [elixirwoes1]: /2017/11/15/an-elixir-gotcha.html
  [elixirwoes2]: /2017/12/02/elixir_deployment_woes.html
  [prolog_facts]: https://twitter.com/AnnieTheObscure/status/961043779871375360
  [web_app_tut]: http://pathwayslms.com/swipltuts/html/index.html
  [clpfd_tut]: http://pathwayslms.com/swipltuts/clpfd/clpfd.html
  [swi-prolog]: http://swi-prolog.org/
  [schedulebot]: https://github.com/braidchat/schedulebot
  [msgpack]: http://www.swi-prolog.org/pack/list?p=msgpack
  [uuid]: http://www.swi-prolog.org/pack/list?p=uuid
  [pengines]: http://www.swi-prolog.org/pldoc/doc_for?object=section(%27packages/pengines.html%27)
  [clpfd]: http://www.swi-prolog.org/man/clpfd.html
