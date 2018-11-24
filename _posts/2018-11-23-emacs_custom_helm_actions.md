---
title: Emacs custom helm actions
layout: post
date: 2018-11-23
---

When I switched from neovim back to Emacs a year or two ago, something that my muscle-memory missed were the key bindings I had for [denite.nvim](https://github.com/Shougo/denite.nvim) for opening files & buffers in splits -- `C-s` to open the result in a horizontal split and `C-v` for a vertical split.
I like `helm` quite a bit as the equivalent of denite in Emacs and it has something like that by default, but it uses `find-file-other-window`, which I don't really like; I find it confusing & somewhat unclear where the new window will actually be and I want complete control over which windows are appearing where.

It took a fair amount of fiddling & reading the source to figure out how to add custom actions to helm, so I thought I'd share what I figured out.
My helm setup stuff can be found in context [here](https://github.com/jamesnvc/dotfiles/blob/master/emacs.d/modules/cogent-helm.el#L50), but I use some weird macro stuff to try to be somewhat lazy, so below is the general idea:

First, we define functions that will open the helm candidates in splits the way we want.
Here, for example, is the function to open files in a horizontal split:

```emacs-lisp
(defun helm-file-switch-to-new--horiz-window (_candidate)
  ;; The candidate to open is passed in to the function
  ;; but we're going to use `helm-marked-candidates' to explicitly
  ;; get the selected list of candidates anyway
  ;; (so the user can use C-SPC to select multiple files & open them all in
  ;; horizontal splits)
  (dolist (cand (helm-marked-candidates))
    ;; focus a new horizontal split...
    (select-window (split-window-right))
    ;; and open the file there
    (find-file cand))
  ;; adjust the windows after opening all those splits
  (balance-windows))
```

We then repeat for vertical splits, replacing `split-window-right` with `split-window-below` to define `helm-file-switch-to-new--vert-window`, then repeat both of those steps with `find-file` replaced by `switch-to-buffer` to make `helm-buffer-switch-to--{horiz,vert}-window`.

This is obviously a lot of boilerplate, which is why in my actual config linked above, I use `cl-macrolet` to define little local macros to generate these repeated function definitions.

Now that we have the functions defined, we can wire them up.
However, it's not quite as simple as just defining a key binding to call our new functions in the appropriate helm map.
Instead, we need to wrap them in an additional function with some helm boilerplate, like so:

```emacs-lisp
(defun helm-file-switch-to-new-horiz-window ()
  (interactive)
  (with-helm-alive-p
    (helm-exit-and-execute-action #'helm-file-switch-to-new--horiz-window)))
;; and the same for the other functions
```

Now, we can finally set up some key bindings to call our functions (I use [general.el](https://github.com/noctuid/general.el), but the equivalent with just `define-key` is straightforward):

```emacs-lisp
(general-def helm-buffer-map
  "C-v" #'helm-buffer-switch-new-vert-window
  "C-s" #'helm-buffer-switch-new-horiz-window)
(general-def helm-projectile-find-file-map
  "C-v" #'helm-file-switch-new-vert-window
  "C-s" #'helm-file-switch-new-horiz-window)
(general-def helm-find-files-map
  "C-v" #'helm-file-switch-new-vert-window
  "C-s" #'helm-file-switch-new-horiz-window)
```

Now you can enjoy quickly & easily opening up buffers from your queries with complete control of how the new windows are being created!
