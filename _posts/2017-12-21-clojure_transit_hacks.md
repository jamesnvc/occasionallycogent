---
layout: post
date: 2017-12-21
title: Disabling Transit Write Caching in Clojure
---

Recently, I've been working on writing another [Braid][] bot, in Racket this time.
I really like making these bots as a way of trying out a new language, as it has enough slightly non-trivial things that I'll be able to get a handle on what the language may be like for bigger things.

The first thing that I usually need to do is figure out how to parse MessagePack-encoded [Transit][transit-format].
Transit is the standard-ish wire format for Clojure that's more efficient than just sending plain EDN and can use either MessagePack or JSON as the underlying protocol.
For Braid, the events that the server sends to the bots are MessagePack-encode Transit and, while most languages will have a library for dealing with MessagePack, most don't have an off-the-shelf solution for Transit.
Hence, the first major task facing me when writing a new Braid bot is building on top of the MessagePack decoding to handle the Transit extensions.

The easiest way to do this is typically to use the extant MessagePack library to turn the Transit data into the native data structures of the language, then post-process it to handle the Transit add-ons (turning strings that start with `~:` in to keywords, vectors that start with `~#u`).
However, I ran in to an interesting problem this time.
I was seeing some places, where I expected to see `~#list`, instead getting `^7`.

It turns out this is Transit's [caching mechanism][transit-cache], which will replace repeated references to tag names and map keys with a reference to said tag or key, instead of having to write it repeatedly.
While I'm sure this is beneficially when sending over a lot of data, with lots of duplicated keys, for this use case it's pretty undesirable.
The only things that repeat are the tags and those are all fairly short.
The real kicker for me though is that determining which symbol the cache code refers to requires knowing the order in which the reader encountered those symbols.
This is because the cache code is just the encoded index of cacheable symbols encountered by the reader -- i.e. the first tag or keyword encountered would be `^1`, the second `^2`, and so on.
This is a problem, because I've already used the MessagePack reader to parse the data into a map, so I don't know which order the keys were encountered in!

I toyed with trying to teach Racket to read the cache, but realized that it was going to be a huge pain in the ass as I'd pretty much have to write my own MessagePack parser.
Furthermore, the other extant bots were probably going to run into this problem (I imagine they already have and this might explain some of the stranger errors I've seen...) and I should probably fix it at the Braid level.
I wanted some way of making the Transit writer not use the write cache.

It seems that there isn't really an easy way so, after digging into the source of `transit-clj` and `transit-java`, I came up with this:

```clojure
(ns braid.server.util
  (:require
    [cognitect.transit :as transit])
  (:import
    (java.io ByteArrayOutputStream)
    (com.cognitect.transit Writer)
    (com.cognitect.transit.impl MsgpackEmitter WriteCache WriteHandlerMap)
    (org.msgpack MessagePack)))

(defn fake-cache
  []
  (proxy [WriteCache] []
    (isCacheable [_ _] false)
    (cacheWrite [s _] s)))

(defn noncaching-writer
  [^java.io.OutputStream out]
  (let [handlers (WriteHandlerMap. ^java.util.Map transit/default-write-handlers)
        packer (.createPacker (MessagePack.) out)
        emitter (MsgpackEmitter. packer handlers)
        write-cache (fake-cache)
        wrtr (proxy [Writer] []
               (write [o]
                 (try
                   (.emit emitter o false (.init write-cache))
                   (.flush out)
                   (catch Throwable e
                     (throw (RuntimeException. e))))))]
    (transit/->Writer wrtr)))

(defn ->transit ^bytes
  [form]
  (let [out (ByteArrayOutputStream. 4096)
        writer (noncaching-writer out)]
    (transit/write writer form)
    (.toByteArray out)))
```

This is pretty much just replicated how `transit-java` creates a writer (which is what `transit-clj` calls), but with the cache replaced with the "dummy" cache created by `fake-cache`, which just says that nothing is cacheable and always just returns the given string as itself (i.e. doesn't turn it into a cache index).

It would be nice if there was some easy way to pass in the cache that the writer uses, or the cache variable wasn't `final`, or something like that, but thanks to Clojure's nicely dynamic nature & excellent Java interop it wasn't too bad to work around.

As a final point, I really hate the way this caching mechanism works.
The way it makes parsing so stateful seems really contrary with how I would expect or want any of this to work.
For the Braid bots I've written in Haskell & Rust in particular, having to maintain state while parsing would make things just so, so much uglier than the fairly nice, declarative way they currently work.
I guess this is really the best way to implement this feature and I appreciate how useful it can be for some applications...but a knob to turn it off would really be nice.

  [Braid]: https://github.com/braidchat/braid
  [transit-format]: https://github.com/cognitect/transit-format
  [transit-cache]: https://github.com/cognitect/transit-format#caching
