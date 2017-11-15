---
layout: post
title: An Elixir Gotcha
date: 2017-11-15
---

A had a bit of a vexing experience with Elixir the other day that I thought I'd document here.
This was in the process of writing [a Braid email bot][emailbot] that lets people view & manage their Gmail inbox in Braid.

I had noticed a few days ago that non-ASCII codepoints were being displayed improperly in the messages that were sent by the email bot.
I was able to see that the bot was printing out the correctly-encoded strings to the iex repl before sending the message to Braid, so I assumed the error was on Braid's end.
This let to a great deal of confusion, as the problem only seemed to manifest when receiving the request:
I could copy-paste the bytes that the Elixir-powered bot retrieved into a Clojure REPL and see that it decoded to the expected [MessagePack-encoded transit][transit], so it seemed like something strange was happening in the Ring handler.
However, after lots of banging my head against the wall, I had no idea what that strange thing could be.

Doing some pair-programming with my co-founder Raf however, got me to look more closely at the bot.
While I could print out the information as retrieved from the Gmail API properly, when I tried to print out the message right before MessagePack-ing and sending, it was garbled.
It transpires that I was using Erlang's `:io_lib.format` when building the message to send and, as the Elixir documentation very helpfully says

> Also note that Erlang’s formatting functions require special attention to Unicode handling.

By searching for the Erlang io_lib documentation, I was able to eventually figure out that the solution was to use the `~ts` format specifier, instead of `~s`.
I'm not entirely sure what the `t` stands for -- "don'**T** break things", I suppose.
Kinda annoyed that the Elixir linters I tried were screaming about the most minor formatting thing, but said nothing about how using this format specifier or `:io_lib` would *seem* to work until they were fed non-ASCII data.

**Summary:** If you're going to use `:io_lib.format/2` in your Elixir code, you probably want to use `~ts`, not `~s`, or it will break when formatting non-ASCII codepoints.

  [emailbot]: https://github.com/braidchat/emailbot
  [transit]: https://github.com/cognitect/transit-format
