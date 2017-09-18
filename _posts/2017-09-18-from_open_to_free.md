---
layout: post
title: "How I Learned to Stop Worrying and Love Free Software"
date: 2017-09-18 18:30:04
---

Something that I started thinking about more recently, inspired by [Bodil][] & [others][], is the difference between Free Software and Open-Source Software.
In the past, I'd been solidly in the Open-Source camp, buying into the idea that it's easier to use, because it lacks the virality of the GPL.

Recently though, as I've become increasing leftist in my philosophy (in response to an increasingly fascist world), I've come around.
I've realized that the advantage of Open Source is that it allows giant companies to use your software, which is fine, but then also build on top of it and not need to give anything back.
As corporations dominate more and more of the world, I'm realizing that the last thing we need to do is make it easier for them to exploit us.

I can see how one may want to have a more permissive license for things like libraries, given that the raison d'être of a library is to be linked in other things and a GPL-esque license on a library then means that the thing using the library must also be GPL'd.
If you just want your generally-useful library to be spread, I can see then using another license.
However for applications, it absolutely makes sense for them to be GPL, since they exist only to be used, so the virality is less of a problem and you don't want someone else to just fork & commercialize something that you've poured a lot of work into.
This is why I was very happy to dual-license (AGPL + commercial) [Braid], since it is a giant monolith that wouldn't really be embedded or linked to something else, but I would not be very happy if some venture-funded dingaling just started building some stuff on top of Braid and selling it.

As I think about it, it seems to me like the more effort that has gone in to something -- which I think would also roughly correspond to how stand-alone the thing is -- the more likely it should be under a Free/Libre license, not Open Source.
I think it's okay for small libraries to be BSD/MIT/Eclipse'd or whatever, since those are the things that will be successful be being used widely, and that big companies will probably just reimplement anyway.

I also think Bodil's point that the idea Open Source comes from ESR is worth considering, given that asshole's libertarian philosophies.
I'm not sure I can be 100% on RMS's side -- I wouldn't go so far as to say that everything needs to be Free/Libre (yet) (note that Braid has a dual license, so we can go for a commercial option) -- but I am definitely not the anti-GPL developer I used to be.

   [Bodil]: https://twitter.com/bodil
   [others]: https://twitter.com/garybernhardt
   [Braid]: https://github.com/braidchat/braid
