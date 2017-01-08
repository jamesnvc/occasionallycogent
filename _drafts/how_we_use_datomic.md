---
layout: post
title: "How We Use Datomic"
---

[Braid][], the new type of chat client that [we've][bloom] been building uses the Datomic database for storage.
We originally started with Datomic because it seemed like a very neat piece of software and fit well with our Clojure-all-the-way-down setup.
After having used it for around a year, I recently did a big refactor of our database layer and how we interact with Datomic.

Originally, we used Datomic in the way we usually use traditional databases -- have a `db` namespace, which has a bunch of functions to wrap calling db functions.
While this was reasonable when using a SQL database (since the db functions themselves are defined in SQL, using [YesQL][] that need a bit of wrapping), it seemed like we were kind of missing out on some of the advantages of Datomic.
Since Datomic's `transact` function takes vectors of normal Clojure data, there seemed to be an opportunity to coalesce how our code communicated with the database.

To this end,

  [Braid]: https://braidchat.com
  [bloom]: http://bloomventures.io
  [YesQL]: https://github.com/krisajenkins/yesql
