---
title: Emacs prettify-symbols-mode in Comments
layout: post
date: 2018-09-25
---

A quick little Emacs thing I figured out that I wasn't able to find from searching, so I thought I'd document it here:

I use the great [PragmataPro](https://www.fsd.it/shop/fonts/pragmatapro/) font for Emacs editor.
One of the cool things about it is the very large set of ligatures for many programming languages (the utility is maybe debatable, but I find them fun -- although I do turn them off when teaching).
The ligatures even include making "`TODO`" markers appear specially.

Emacs on Linux, however, doesn't seem to support OpenType ligatures natively.
The solution is to use `prettify-symbols-mode` to have the character combinations rendered as the special character that would be used as the ligature.
This works well enough, with the additional nicety of it being more flexible with exactly which ligatures you get & making it easy to disable them.

However, it seems that by default, prettify-symbols won't prettify symbols in comments or in strings.
This is reasonable, I suppose, but it makes the `TODO`, `HACK`, `BUG`, etc ligatures fairly useless.
I finally decided to try to figure out how to make symbols in comments also get prettified.

I eventually figured out that `prettify-symbols-mode` uses a predicate to check if the region should be prettified & makes this configurable.
The [default predicate](https://github.com/emacs-mirror/emacs/blob/ee3e432300054ca488896e39fca57b10d733330a/lisp/progmodes/prog-mode.el#L102) makes what  seem like reasonable checks, but has one key & non-obvious bit:

```emacs-lisp
(defun prettify-symbols-default-compose-p (start end _match)
  ...
  (not (or (...)
           (...)
           (nth 8 (syntax-ppss))))) ;; <- what's this do?
```

Reading the docs for `syntax-ppss`, it transpires that that little bit is checking if the region is a comment.
I was able to just set the `prettify-symbols-compose-predicate` to be [my own version of that function](https://github.com/jamesnvc/dotfiles/blob/master/emacs.d/modules/cogent-pragmata.el#L216) without that last check and now I get my fancy ligatures in comments!

{% include image.html url="/images/emacs_ligatures/todo_ligature.png" caption="TODO ligature in a comment" alt="Emacs with a TODO ligature" %}


[See my PragmataPro emacs config here](https://github.com/jamesnvc/dotfiles/blob/master/emacs.d/modules/cogent-pragmata.el)
