---
layout: post
title: "I Don't Want to be Static"
date: 2017-06-06 10:48:25
---

Most of the work I've been doing the last few years has been in Clojure.
I really enjoy the language, but I always like trying to experiment with new things and challenge myself.
To that end, the last few side projects ([Braid][braid] bots) have been written in different languages -- [two][giphybot] [bots][octocat] in Rust, [one][greeterbot] in Haskell, and [one][emailbot] in Elixir.

 I wanted to try using Rust and Haskell because I've been using more dynamically typed languages the past few years and I wanted to gain a better appreciation of strong type systems.
 Haskell in particular is a language that I've been interested in on-and-off since high school and really wanted to finally wrap my head around.

In the midst of writing the bot in Haskell, I actually managed to have a friendly and productive conversation about typing on the Internet -- specifically, on Mastodon.
I was feeling like I was missing something when it came to strong type systems:
I had just finished reading a paper on [profunctor optics][profunctor] and was feeling really interested in the power of type systems, but I couldn't quite grasp what they enabled vis-à-vis writing software.
In that spirit, I vented a little on Mastodon about how the strong type systems seemed more of an encumbrance than an assistance to me, especially coming from Clojure (I complained about the overwhelming syntax a bit too).

I got some good responses back and the dialogue I had with them seemed to establish that, for most people, the advantage of type systems is reducing error rates.
While this is somewhat obvious, I found it disappointing.
In my experience, the challenging part of making software is growing & adapting it to changing circumstances -- while errors happen, they are generally straightforward enough to deal with and not really my primary concern day-to-day.
Those errors that do become things that I'm concerned with are very rarely the type of bugs that type systems would catch anyway, tending to be more along the line of performance problems and network issues.
With all the additionally complexity of strong type systems and the additional hoops they require one to jump through, I was hoping that there was some way it allowed greater adaptability, rather than less.

I suppose a lot of why one would choose a stronger/more static versus a "weaker"/more dynamic type system has a lot to do with the sort of work one is doing.
If the things I was working on had a much more locked-down & well-understood problem domain and my primary concerns were slow, incremental change, I suppose the calculus would favour a language that let me "freeze" the structure of the code more.
However, most of the things I work on begin as experiments and change drastically and continually; in those domains, languages like Clojure (and maybe Elixir in the future, as I'm liking the way it works a lot) are what I'll turn to.
I feel like they allow code to evolve & stay fluid much longer and more easily.

I will be keeping my eye on the strong-typing family & continuing to experiment -- my opinions on how I like to get things done has changed a lot in the past, so some day I may come to favour them -- but for now, I remain decidedly in the camp of dynamic languages.

  [braid]: https://braidchat.com
  [giphybot]: https://github.com/braidchat/giphybot
  [octocat]: https://github.com/braidchat/octocat
  [greeterbot]: https://github.com/braidchat/greeterbot
  [emailbot]: https://github.com/braidchat/emailbot
  [profunctor]: https://arxiv.org/pdf/1703.10857v1.pdf
  [asked]: https://mastodon.social/web/statuses/5394625
