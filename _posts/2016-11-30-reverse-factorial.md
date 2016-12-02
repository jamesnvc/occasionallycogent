---
layout: post
title: "Reverse Factorial"
date: 2016-11-30 22:00:00
categories: clojure clojurescript
---

I recently came across [this post](http://re-factor.blogspot.ca/2016/11/reverse-factorial.html) about implementing a "reverse factorial" in Factor.

The idea of the reverse factorial is, given a number `x`, find the number `y` such that `y! = x` or false if no such integer exists.

The Factor solution computes the reverse factorial by computing factorials of 1, 2, ... until the factorial is equal to the number in question or is greater (in which case the answer is `false`).

I decided to try to do the solution another way: Instead of creating increasing factorials, repeatedly divide the number by increasing integers until the number is one (in which case, the last divisor is the reverse factorial) or the number doesn't divide equally (in which case, the answer is `false`).
Implementing it also let me use a neat Clojure feature, the [`reduced` function](http://clojure.github.io/clojure/clojure.core-api.html#clojure.core/reduced).

{% highlight clojure %}
(defn reverse-factorial
  [n]
  (reduce
    (fn [n div]
      (cond
        (= 1 n) (reduced (dec div))
        (integer? (/ n div)) (/ n div)
        :else (reduced false)))
    n
    (drop 2 (range))))

(= (reverse-factorial 3628800)
   10)

(= (reverse-factorial 479001600)
   12)

(= (reverse-factorial 6)
   3)

(= (reverse-factorial 18)
   false)
{% endhighlight %}
