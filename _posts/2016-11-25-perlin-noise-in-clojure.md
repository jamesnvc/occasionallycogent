---
layout: post
title: "My Sister's Birthday, Perlin Noise, and Ancient Mammals"
date: 2016-11-25 22:00:00
categories: clojure clojurescript
---

A few years ago a bought a [domain name for my sister][sadie] for her birthday.
Every year since, I've been making a sort of interactive birthday card there for her and in the process, teaching myself something new.

Last year, I made a little app that would display videos from a few of her favourite k-pop bands, using [om.next][omnext] (since that was something we'd been considering using at work).

This year, I decided to do something a little more interesting, both for me and her and make something actually interactive.
I was thinking that it eventually could even be a game, time allowing (maybe next year?).

To start, I was thinking I'd have a picture of a [baluchitherium][baluch][^1] on the page that one could move around with the arrow keys (and later, could be like, dodging obstacles or whatever).
One of the things I've been trying to practice this year is being comfortable drawing stuff, so I used the fantastic pixel art tool [aseprite][] to draw an animated gif of a baluch:

{% include image.html url="https://bytebucket.org/jamesnvc/sadie-birthday-2016/raw/709558c84071f76e73a7b5a43162d6386fabe586/art/baluch.gif" alt="baluch" caption="A majestic flying baluch" %}

It's simple enough to put this on the page, the use the arrow keys to move it around.
This was probably good enough for a birthday card, but I wanted to do something more challenging...

## Perlin Noise

I had read a great post by [eevee about Perlin noise][eevee] not too long ago and thought it was interesting, but I didn't full get how the algorithm worked.
The best way to understand an algorithm is to implement it, so I decided I'd make the background that the baluch was flying over be Perlin noise generated on-the-fly.

I started by trying to do a straightforward port of [eevee's python implementation][perlinpy] to [ClojureScript][perlincljs].
It took me a while to get things working properly, but it was mostly a mechanical transformation of Python to ClojureScript (it was pretty fun to translate mutating loops into reduces and such though).

The tricky part was dealing with the fact that the algorithm was quite computationally expensive - it takes a significant amount of time to generate each frame.
Since this is running in the browser, which only executes Javascript on a single thread, it would block constantly, so you wouldn't be able to move the baluch around at all!

This obviously would not stand, so I had to find a way around this.
I didn't feel like trying to optimize the algorithm (although that would definitely be a good thing to try to do), so I needed to somehow move the computation off the main thread.
I first thought I could just use [`core.async`][async] (which is a great library), but realized quickly that that wasn't the right approach -- `core.async` is great for asynchronous I/O, but it's still Javascript, meaning while it can do concurrency, it still isn't parallelism!

Luckily, there's a (relatively) new Javascript API for [WebWorkers][webworkers] that actually allows Javascript to run on multiple threads!
Reading how it worked, it seemed like it was going to be kind of complicated to use, especially since I wasn't writing Javascript directly, but compiling from ClojureScript.

Fortunately, there's a solid ClojureScript library called [servant][] that wraps WebWorker stuff and makes in fairly easy to have some ClojureScript code run in a background thread!
I remainly slightly unclear about how data gets passed in and out of the `servant` functions, but eventually I got things working by just passing in primitive data and encoding the return values of the functions using [`transit`][transit].

The background updates fairly infrequently (depending on how fast your browser can render each frame), but it works!

{% include image.html url="/images/perlin/site.png" alt="site screenshot" caption="Check out that sweet random -- but smooth! -- noise" %}

Check out the [site][sadie][^2] (and use the arrow keys to fly the baluch around) and have a look at the [code][source].

  [sadie]: http://sadiemae.rocks
  [omnext]: https://github.com/omcljs/om/wiki/Quick-Start-(om.next)
  [baluch]: https://en.wikipedia.org/wiki/Paraceratherium
  [aseprite]: https://www.aseprite.org/
  [eevee]: https://eev.ee/blog/2016/05/29/perlin-noise/
  [perlinpy]: https://gist.github.com/eevee/26f547457522755cb1fb8739d0ea89a1
  [perlincljs]: https://bitbucket.org/jamesnvc/sadie-birthday-2016/src/709558c84071f76e73a7b5a43162d6386fabe586/spacebaluch/src/cljc/spacebaluch/perlin.cljc?at=master&fileviewer=file-view-default
  [source]: https://bitbucket.org/jamesnvc/sadie-birthday-2016/src/master/spacebaluch
  [async]: https://github.com/clojure/core.async
  [webworkers]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers
  [servant]: https://github.com/marcopolo/servant
  [transit]: https://github.com/cognitect/transit-cljs

  [^1]: Long story, but basically we have a family tradition of everyone having an animal, and for some reason my mum's branch of the family has weird animals - my sister and one of my brothers are baluchitherium, these weird prehistoric creatures. I happen to be a pterodactyl.
  [^2]: Probably don't leave it open too long though - it will devour as much CPU as it can to push out the noise.
