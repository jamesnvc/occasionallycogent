---
layout: post
title: "Politics in Programming Pedagogy"
date: 2018-02-21
---

A few months ago, I saw [this great article][article] about teaching programming with examples that deal with real issues (which I highly recommend reading).

I was very inspired by this, as it seemed like a way to combine one of my beliefs -- that there are no apolitical acts and that by saying nothing, you reinforce the status quo -- and something that I regularly do -- teach iOS programming at a coding bootcamp.

Today, as part of an overhaul of the curriculum, I gave lecture that I hadn't before, about databases (specifically, using the C Sqlite API in Swift).
It may be difficult to figure out how to make more abstract things relevant, but the article that put the idea in my head was about precisely this, so I had no excuse.

For the last year or so, [my partner][amanda] has been learning a lot about educating vis-à-vis the perspectives, experiences, and issues of Indigenous peoples, with a focus on the Anishinaabe, Haudenosaunee, and Cree, whose territory is what is also known today as Ontario.
One of her first attempts at decolonizing her own pedagogy was last April, where she guided 6 and 7 year old children through learning about the inequities in potable water access in many First Nation communities.
I was struck by realizing how prevalent these issues were and that how even these young children pretty quickly understood the magnitudes of this injustice.

With this in mind, I devised a plan for the lecture to use information about First Nation communities with long-term boil water advisories as the data we'd be populating our database with.

First I taught the class how to make a table to store this information, then led them through writing queries to find the number of communities that had not had clean water in a given number of years.
We saw that "a given number" could mean *twenty-three years* in the case of Neskantaga.

I think that teaching like this was beneficial in a couple ways.

Firstly, there's the obvious that it sheds light on issues impacting rural First Nation communities.
There are several international students in this cohort and I think it's good that they maybe learn things about Canada beyond what happens is Toronto.
There are also students who have lived their whole lives in Ontario without a clue that this was even happening.
By making more people aware of the inequities in access to basic needs that exist here and now, we can create an environment where it isn't politically possible to continue sweeping these issues under the rug.

In addition, students were more attentive to the content that was being taught.
I think it caught them off guard because it was an unexpected departure from the usual swing of things.

I hope too that they maybe consider that programming isn't just writing apps and that it can have to do with real problems in the world...and maybe some of them will be inspired to try to address those problems.

<p class="parabreak"></p>

I said above that I haven't introduced culturally relevant ideas in my other lectures, because it would be "kind of challenging".
It being challenging isn't a reason not to do it though; moving forward, I am going to make an effort to try to do something in every lecture I do.

If you're interested in the code, it is available [here][lecture_code] (the interesting files are probably [Database.swift][dbswift], which defines all the database functions used, and [ViewController.swift][vcswift], where the functions are called)

I would love to hear feedback from educators about this or advice as to other things I could integrate in my teaching -- all my contact info is below.

  [article]: http://archive.oreilly.com/pub/a/oreilly/news/feuerstein_1000.html
  [amanda]: https://twitter.com/amandasantos_ed
  [lecture_code]: https://bitbucket.org/jamesnvc/lhl-w6d2-2018-02-20/src/2c88cec85650295ea772717c646200d59eec5375/LiteDbDemo/?at=master
  [dbswift]: https://bitbucket.org/jamesnvc/lhl-w6d2-2018-02-20/src/2c88cec85650295ea772717c646200d59eec5375/LiteDbDemo/Database.swift?at=master&fileviewer=file-view-default
  [vcswift]:         https://bitbucket.org/jamesnvc/lhl-w6d2-2018-02-20/src/2c88cec85650295ea772717c646200d59eec5375/LiteDbDemo/ViewController.swift?at=master&fileviewer=file-view-default
