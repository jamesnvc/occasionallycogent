---
layout: post
title: Small Teams
date: 2017-11-11
---

Something that the last few years of building things with Lean Pixel and then Bloom has really reinforced in me is a belief in the power of small teams.
With a team as small as two people, we've managed to build some things that, in my completely unbiased opinion, are pretty impressive.
We built Rookie, with a functioning in-browser video editor.
We made Braid, a complete reimagining of what a chat program can be, that we've been using exclusively for years now.

The thing that I really appreciate about both these products though, is that they've remained focused and relatively well-factored (we are in fact currently in the process of refactoring Braid to have a more extensible architecture).
This is the real value of something built with a small team -- it remains understandable and still exhibits a coherent vision and unity of design philosophies.

A key aspect to the small-team nature is that people are allowed to stop and ruminate for a while.
By being able to take a week to think before jumping to implementation, we're able to have things structured in a way that is more harmonious with the existing overall design, instead of just jumping in, immediately writing code, and having to pile hacks on top of hacks.

This is the great tragedy of large, venture-funded companies.
They need to spend this money they've been given, so they start hiring more people.
There's this initial burst of productivity, as more people lets them write more code.
Inevitably though, this starts to collapse under it's own weight:
The system grows so complex and creaking that it threatens to become unsustainable.
Unfortunately, the way companies usually react to this looming collapse is to just throw more bodies at it, and hire even more people to try to push things through faster or prop up crumbling infrastructure.

The most salient example of this is Facebook.
Last year, there were some interesting "teardowns" of the Facebook iOS app.
It revealed that the massive, 250MB binary was due to absolute mountains of duplicated code:
Thousands of classes, most of them unused, many of which were reimplementing the same functionality, because things have become so complicated and murky that no-one knows what someone else has already done.
Multiple copies of giant frameworks linked in for the same reason.
Their system has grown so large that it defies the understanding of any one person there.

The way they react to this issue is the worst part.
There was a talk that some Facebook engineer gave, describing how git was failing them.
They keep all their code in one monorepo (which I think is reasonable), which has grown to something like a terabyte of history (which I do not).
Their solution is not to reconsider what has lead them to this place, but to try to pile more work on top of the problem and make git able to deal with a ludicrously massive repo (because if any software needs more complexity added to it, it's git).

I think that this is absolutely the wrong way to build things, but also a natural consequence of the economic system that most of these companies are labouring under.
If you're venture funded, you need to spend the money, so you need to hire people, so you end up with this sort of debacle.

There are absolutely times, especially early on in the lifecycle of a project, when having more hands is useful.
However, in my experience, this is the perfect opportunity to bring on interns and actually provide them with useful training.
By teaching a new developer the stack you're using in a tabula rasa project, they can actually learn how the stack works, instead of having to wade through the complexities and historical cruft of some legacy project that is foisted upon them.
By giving them a relatively free hand to design things, they can actually learn architectural skills that normally they have no good way of picking up.

It's a perfect scenario for everyone:
The person being trained learns more about the tools and how to think about things at a whole-system level, developing the skillset of a much more senior developer in a relatively short amount of time.
Meanwhile, they free up the more senior developers from the monotony of implementing all the low-hanging fruit that a new project needs taken care of, letting them focus on the more complicated parts immediately.
Then, by the time the project is reaching a steady-state of relatively small changes, or changes that require careful deliberation and are best suited to small teams, the intern's term is up.
They return to school or move on to a permanent job, having gained a massive amount of extremely valuable real-world experience.
Most importantly, the team doesn't have people sitting around, looking to make work for themselves.
They're able to stay small and focused, with everyone understanding the existing system, and everyone has well-designated and vital roles.

This may not be a perfect fit for all environments or teams, but I think it is a vastly more sustainable one.
