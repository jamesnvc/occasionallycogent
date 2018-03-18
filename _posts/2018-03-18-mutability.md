---
layout: post
title: What Does Mutability Mean?
date: 2018-03-18
---

When teaching programming, something that tends to come up a lot is when and how values can change.
While there is a lot of jargon (values, references, mutability) to make communicating with people that already understand these ideas easier, it can be difficult to explain to people that don't already understand the ideas.

The below was initially written to explain to a co-worker learning Clojure what we mean when we say "values are immutable" and I thought maybe someone else could find it useful.

## Mutability

In Clojure, values are immutable by default.
If you have `(let [foo [1 2 3]] ...)` you can't change that list.
You can make `foo` refer to a new value, but you can't change the value itself.
For example, if you have `(let [foo [1 2 3] bar foo] (conj foo 4))` then the values of `foo` & `bar` are still the same, `[1 2 3]` - the `conj` function returns a new vector (that we haven't given a name to).
Even if you do something like

```clj
(let [foo [1 2 3]
       bar foo
       foo (conj foo 4)] ...)
```

`foo` now refers to the vector `[1 2 3 4]`, because we've "shadowed" what `foo` refers to, but `bar` is still `[1 2 3]`.

In languages that have mutable values, for example Python, you could do:

```python
foo = [1, 2, 3]
bar = foo
foo.append(4)
```

Then `foo` would be `[1,2,3,4]` and so would `bar`, because the list is mutable, so you can actually change the underlying value the names refer to.
In Clojure, even with atoms, only the atom itself is mutable, so, for example:

```clj
(def foo (atom [1 2 3]))
(def bar @foo)
(swap! foo conj 4)
```

Now foo is an atom, `@foo` is `[1 2 3 4]` but bar is still `[1 2 3]`

This makes things much less confusing & reduces the potential for bugs, because values aren't changing out from under you.
In the python example, imagine `foo` is getting passed to some other function, then that could make the value of your `bar` variable change!
