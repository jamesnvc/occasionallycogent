---
layout: post
title: "How We Use Datomic"
---

[Braid][], the new type of chat client that [we've][bloom] been building uses the [Datomic][] database for storage.
We originally started with Datomic because it seemed like a pretty neat piece of software and fit well with our Clojure-all-the-way-down setup.
After having used it for around a year, I recently did a big refactor of our database layer and how we interact with Datomic.

# What We Did

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
                     (map (comp ::return meta))
                     (remove nil?))]
    (map #(% tx-result) returns)))
{% endhighlight %}

The final thing I wanted to add was having `run-txns!` do some validation, taking advantage of the fact that there is now only one place where `datomic.api/transact` is called.
Similarly to how I handled return values of transactions, my idea was to have a "check" function in the metadata of a transaction.
Then, `run-txns!` could take advantage of a neat Datomic function, [`with`](http://docs.datomic.com/clojure/index.html#datomic.api/with), which gives you a database that would be what happened if you evaluated a transaction, without actually running said transaction.

The implementation now looks like this:
`BLOOP
{% highlight clojure %}
(defn extract
  "Like `map` but filter out nil results"
  [f & colls]
  (remove nil? (apply map f colls)))

(defn run-txns!
  [txns]
  (let [[returns checks] ((juxt (partial extract ::return)
                                  (partial extract ::check))
                            (map meta txns))]
      (when (seq checks)
        (let [test-db (d/with (db) txns)]
          (try
            (doseq [check checks]
              (check test-db))
            (catch AssertionError e
              (throw (ex-info "Transaction Failed"
                              {::error (.getMessage e)}))))))
      (let [tx-result @(d/transact conn txns)]
        (map #(% tx-result) returns))))
{% endhighlight %}
BLOOP`

And we can now write transactions that validate the results as follows

`BLOOP
{% highlight clojure %}
(defn bot-watch-thread-txn
  [bot-id thread-id]
  [^{:braid.server.db/check
     (fn [{:keys [db-after]}]
       (assert
         (= (get-in (d/entity db-after [:bot/id bot-id]) [:bot/group :group/id])
            (get-in (d/entity db-after [:thread/id thread-id]) [:thread/group :group/id]))
         (format "Bot %s tried to watch thread not in its group %s" bot-id thread-id)))}
   [:db/add [:bot/id bot-id] :bot/watched [:thread/id thread-id]]])
{% endhighlight %}
BLOOP`

# Why We Did It

So, what did we gain from all this?

For one, our transaction functions are now all declarative, which I think fits better with the philosophy of using immutable data structures.
Instead of all our database functions performing arbitrary state changes, we isolate state changes to one function, `run-txns!`.
Also, because of this isolation, we have one good spot in which to perform validation.

The biggest reason though, was that it was fun!

  [Braid]: https://braidchat.com
  [bloom]: http://bloomventures.io
  [Datomic]: http://datomic.com/
  [YesQL]: https://github.com/krisajenkins/yesql
