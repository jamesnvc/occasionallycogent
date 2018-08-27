---
title: Writing an HTTP/2 Client in Prolog
layout: post
---

A few weeks ago, I had a modest goal, which kind of spiralled out of control and ended up with [a PR to SWI-Prolog](https://github.com/SWI-Prolog/packages-ssl/commit/964a8b2132d77a6689f8e8ba849b86b9888f0aed) and [my most ambitious Prolog library yet](https://gitlab.com/jamesnvc/prolog_http2_client).

I was building out a little app idea for a friend of mine and I decided that, since this is a for-fun project that I'm the only one working on, I'd write the server backend in [my current favourite language](/2018/03/24/discovering_an_old_language.html), Prolog.
Without going in to the details of exactly what the app is, the backend just needs to monitor a feed for changes & send push notifications to iOS devices.

The monitoring part was simple, but when I started looking at how to send push notifications, I was slightly stymied.
While Apple has some documentation how to send notifications, I found it surprisingly challenging to find example code of how to send notifications; it seems like most people use some service to do push notifications & don't implement it themselves.
Using something like SNS or similar is probably the most reasonable approach for a real production thing, I wanted to just do it myself -- why take on more dependencies on some other service, especially when sending notifications is really all this server needs to do?

However…it seems like that, while Apple's push notification service used to work by the server opening a plain ol'TLS-over-TCP socket & writing data, it now uses HTTP/2.

Now, I'd heard of HTTP/2 before now, but I didn't really know much about it…I'd heard it could reuse connections or something, but otherwise was basically the same thing.
However, this is really not accurate.
HTTP/2 is a binary format, that has a whole complicated way of breaking "streams" into "frames" and multiplexing those streams in one connection.
The point being, this wasn't something I could just use SWI-Prolog's http library and sort of hack around -- this is something completely different.

This is the point where the reasonable thing is *definitely* to just use an existing service.
But…if HTTP/2 is the future, then someone needs to implement it for SWI-Prolog, so why not me?
So, I buckled down, pulled up [a](https://httpwg.org/specs/rfc7540.html) [few](https://httpwg.org/specs/rfc7541.html) [RFCs](https://www.rfc-editor.org/rfc/rfc7301.txt) and got to work.

The [HTTP/2](https://gitlab.com/jamesnvc/prolog_http2_client/blob/master/prolog/frames.pl) and [HPACK](https://gitlab.com/jamesnvc/prolog_http2_client/blob/master/prolog/hpack.pl) (the encoding HTTP/2 uses for headers) parsers were pretty fun to write.
I really love Prolog's [DCGs](https://www.metalevel.at/prolog/dcg) and being able to write one piece of code that can both serialize & deserialize is just so cool.
I had to do a fair bit of learning & thinking to be able to do so, using things like the [`delay` pack](https://github.com/mndrix/delay) and the extremely cool [`reif` library](http://www.complang.tuwien.ac.at/ulrich/Prolog-inedit/swi/reif.pl), but I am very pleased with how the results look.

The client itself, I'm not quite as pleased with.
It's more-or-less functional now, but definitely has some weird edge cases that need to be figured out, not the least of which is actually having some degree of robustness & error handling (contributions welcome!).

The other missing piece I needed is something called TLS-ALPN; "Application-Layer Protocol Negotiation", an extension to TLS to allow the negotiation on what the protocol the application-level communication will be using.
This is something that new OpenSSL library provides, but does happen as part of the SSL handshake.
So, if I wanted to make use of it, I would need to get down to that level.

SWI-Prolog's SSL support is provided via OpenSSL, using SWI's pretty great FFI.
I haven't done C in quite some time, but the existing code was quite readable and so, with lots of [useful feedback](https://github.com/SWI-Prolog/packages-ssl/pull/130), I was able to fairly easily add the requisite functionality to SWI-Prolog.
I really love how straightforward it was to add this functionality and how nice the Prolog maintainers were to someone just showing up out of the blue with a patch.

I had a lot of fun writing this library and I hope that others find it useful, even if just as a reference.
If someone out there wants to help make this library more robust & production ready, please [check it out!](https://gitlab.com/jamesnvc/prolog_http2_client).
