---
layout: post
title: "How We Use Datomic"
---

[Braid][], the new type of chat client that [we've][bloom] been building uses the [Datomic][] database for storage.
We originally started with Datomic because it seemed like a pretty neat piece of software and fit well with our Clojure-all-the-way-down setup.
After having used it for around a year, I recently did a big refactor of our database layer and how we interact with Datomic.

Originally, we used Datomic in the way we usually use traditional databases -- have a `db` namespace, which has a bunch of functions to wrap calling db functions.
While this was reasonable when using a SQL database (since the db functions themselves are defined in SQL, using [YesQL][] that need a bit of wrapping), it seemed like we were kind of missing out on some of the advantages of Datomic.
Since Datomic's `transact` function takes vectors of normal Clojure data, there seemed to be an opportunity to coalesce how our code communicated with the database.

My goal was to refactor things so that instead of all the database functions directly calling `transact`, they would return transaction data, that would be passed into one wrapper function.

A trivial example of the transformation:
{% highlight clojure %}
; old-style
(defn set-user-password!
  [conn user-id password]
  @(d/transact conn [[:db/add [:user/id user-id]
                      :user/password-token (password/encrypt password)]]))
; new-style
(defn set-user-password-txn
  [user-id password]
  [[:db/add [:user/id user-id] :user/password-token (password/encrypt password)]])

; new-style invoked like
(run-txns! (set-user-password-txn user-id passwd))
{% endhighlight %}

The `run-txns!` function, for now, is just a simple wrapper over `datomic.api/transact`, supplying the connection argument.

Using this style, we can now do things like logically group database alterations together in one transaction (by `concat`-ing the various `*-txn` functions) and we now have one place to validate things.

However, there are a few functions that this simple refactor is unable to handle; namely, functions that need to do something with the result of the transaction.
In our codebase, the main offenders are our various `create-*` functions, which insert a new entity and return said entity, using the `create-entity!` helper function:

{% highlight clojure %}
(defn create-entity!
  "create entity with attrs, return entity"
  [conn attrs]
  (let [new-id (d/tempid :entities)
        {:keys [db-after tempids]} @(d/transact conn
                                      [(assoc attrs :db/id new-id)])]
    (->> (d/resolve-tempid db-after tempids new-id)
         (d/entity db-after))))
{% endhighlight %}

The solution I came up with was using Clojure's metadata facilities to indicate that a particular transaction wanted to return something.
Then, `run-txns!` can gather all said functions and return the result of giving all of them the created database.

`create-entity!` now becomes the following:

{% highlight clojure %}
(defn create-entity-txn
  "create entity with attrs, return (after-fn entity)"
  [attrs after-fn]
  (let [new-id (d/tempid :entities)]
    [(with-meta
       (assoc attrs :db/id new-id)
       {:braid.server.db/return
        (fn [{:keys [db-after tempids]}]
          (->> (d/resolve-tempid db-after tempids new-id)
               (d/entity db-after)
               (after-fn)))})]))
{% endhighlight %}

(the `after-fn` parameter is there because the various `create-*!` functions want to call a `db->*` function to change tho returned entity into a map in the format Braid expects to use)

To make this work, `run-txns!` now looks like this:
{% highlight clojure %}
(defn run-txns!
  [txns]
  (let [tx-result @(d/transact conn txns)
        returns (->> txns
                     (map (comp :return meta))
                     (remove nil?))]
    (map #(% tx-result) returns)))
{% endhighlight %}

  [Braid]: https://braidchat.com
  [bloom]: http://bloomventures.io
  [Datomic]: http://datomic.com/
  [YesQL]: https://github.com/krisajenkins/yesql
