---
layout: post
title: The Whys and Whats of Braid
date: 2018-03-28
---

Perhaps the most challenging part of running a remote business is communication.
There is so much that is taken for granted with casual spoken conversation that translates very poorly to electronic communication.

[We][bloom] have been working almost 100% remotely for the last few years and we've built a tool that has done a lot to make that possible -- our chat program, [Braid][mainbraid].

When we first started, we were using existing chat programs very heavily (initially HipChat and then later Slack).
However, in the summer of 2015, Raf had an idea for a new kind of chat client, one that made conversations the central concept.

As most people that have used traditional chat clients have experienced, it can be pretty frustrating when you have more than one conversation going on simultaneously, as the replies get all intermingled and it's hard to tell who is talking to whom.

{% include image.html url="/images/why_braid/bad_chat_mixed.png" alt="Chaotic intermingled chat room" caption="Who's talking to who?" %}

Even worse is when someone has a question that can't be responded to for some amount of time -- unless the chat room is completely dead, it will have been pushed off the screen and forgotten by the time the answer is ready.

{% include image.html url="/images/why_braid/bad_chat_new.png" alt="Chaotic intermingled chat room" caption="Poor NewUser will never get an answer..." %}

Braid solves these problems by replacing the traditional model of the chat room as one intermingled conversation on the given topic, with any number of stand-alone threads that can be "tagged" with the relevant topics.
Because these threads are all separate, there can be any number of threads with the same tags (e.g. `client`, `bug`, `funny`) but the conversations stay distinct.

{% include image.html url="/images/why_braid/braid_multi_tags.png" alt="braid threads" caption="Braid can have any number of threads with any number of tags" %}

An unintended consequence of this approach is that it lets you clear out your chat the way you can clear out your email inbox.
Because threads re-open if someone else responds (although you can mute a thread if it's busy & irrelevant to you), after you've read a thread & replied you can just close it, leaving you with a nice clean slate.
It seems like a small thing, but the difference between your chat client only showing what's new and your chat client constantly showing a bunch of maybe-relevant *stuff* is more calming than you might think.

{% include image.html url="/images/why_braid/braid_empty_inbox.png" alt="braid empty inbox" caption="Nothing is as calming as knowing everything is taken care of" %}

One of the important design principles of Braid is that it shouldn't be intrusive in your life.

That's why it's designed so that you can function perfectly well only checking Braid once in a while and don't need to constantly monitor what's happening -- because threads are separate, you can easily catch up on what's important and ignore what's no longer relevant.
While Braid has notifications, they are all turned off by default and we think that people should try getting by without them.
It may seem tough at first, but it's pretty freeing!

This is also why, although Braid has [a mobile interface][braidmobile], we use it rarely (if you'd like to help improve it though, [contributions are welcome!][braidgithub]).
Braid is for getting work done and we don't want that work to bleed over in to the rest of your life.
We aim to let you communicate effectively at work but also be able to step away and enjoy the time when you're not in front of a computer.

We'd love for you to [give Braid a try][trygroup], maybe join the [Braid development chat][braidgroup], and [have a look at the code][braidgithub]!

  [bloom]: https://bloomventures.io
  [mainbraid]: https://braidchat.com
  [braidmobile]: https://m.braid.chat
  [trygroup]: https://braid.chat/try
  [braidgroup]: https://braid.chat/braid
  [braidgithub]: https://github.com/braidchat/braid
