---
layout: post
title: One Week With the Model 01
date: 2017-10-30
---

It's been one week now since [my Keyboardio Model01 arrived][arrived]!
At this point, I would say that it's been worth the two and a half year wait.

## Hardware

The hardware itself is extremely nice.
Jesse and Kaia just [posted today][backer_update] about the troubles that they've been facing with the wood enclosures, but their QA game has definitely been on point -- my unit is gorgeous and looks perfect.
The natural texture of the wood is beautiful and feels delightful.

{% include image.html url="https://i.imgur.com/woMRY6I.jpg" alt="keyboard on desk" caption="My new keyboard setup" %}

Typing on the unit has been taking a little bit of getting used to.
The keyboard I used before was a standard keyboard, so the split ergonomic design is completely new to me.
I adjusted to most of the letter keys pretty quickly (I hit 75 WPM after a few days (my best on my old keyboard was ~100)), with a few exceptions -- I keep confusing the keys labelled `b` and `v` on QWERTY (I'm using a Dvorak layout, so `x` and `k`, respectively) as well as `v` and `c` (`k` and `j` for me).
The problem I'm having with these keys is that I'm reaching too far though, so as I get more used to having the keys in what's actually an easier place to hit, the incidences of those errors has been steadily decreasing.

The thing that's been taking the longest to adapt to is having the modifier keys on the thumbs.
Since I have no sort of muscle memory to adapt, it's purely a matter of memorizing where these new keys are, which slows me down some -- instead of just hitting control/super/shift I need to thinking about which key to use (and similarly for the backspace, space, and escape keys (I've remapped the key labelled `Alt` to Escape)).

One other thing I'm still figuring out with this keyboard is which finger is optimal for each given key.
As promised, the individual shaping of the keycaps does a great job of guiding the fingers to the keys and typing prose on it feels great and naturally encourages me to use the "proper" fingering.
However, the center column of buttons, in particular the Tab and Enter buttons are something I'm not sure about.
It initially seemed like the obvious way to hit those keys was to stretch my index fingers, but that actually started to feel fairly uncomfortable.
I'm currently trying to use my thumbs instead; it seemed weird, but it actually feels very comfortable and again, the shape of those keys actually seems to guide the thumbs in perfectly.

## Firmware

As alluded to above, I've done a bit of remapping of the default firmware.
I've been using a Dvorak layout for a number of years now, so I knew I was going to have to remap to do that, but the power of the Kaleidoscope firmware allowed me to do so much more.

{% include image.html url="https://bytebucket.org/jamesnvc/keyboardiolayout/raw/12e14d9ff2f2d39f1e4feded87f37da8f4cc30d5/docs/keyboard-layout.png" alt="my keyboard layout" caption="<a href='https://bitbucket.org/jamesnvc/keyboardiolayout/overview'>My keyboard layout</a>" %}

Some highlights of what I've been able to do:

### OneShots

[OneShot][] modifiers and layer keys let me tap a modifier then the other key instead of needing to hold down to chord -- e.g. instead of contorting my hand to hit Alt-X, I tap the Alt, let go, then tap X, and the operating system detects it like I pressed the key combination as normal.
This goes a very long way to reducing weird hand contortions and has already become something I wouldn't want to use a computer without.

One-shots also enable one to double-tap a key to "lock" it on until you press it again.
This is useful for doing something that requires you to keep, for example, the control key held down for a while, or hit multiple modifiers at the same time.
It is especially useful for layer switching; I have the palm key set as one-shot layer switches, so I can double-tap the left palm key and the "HTNS" keys become arrow keys.

### SpaceCadet

The [SpaceCadet][] plugin is something I'm very happy to have.
Simply, it makes the Shift keys act as normal when held in conjunction with other keys, but when tapped act as left and right parenthesis (most of my work is in Clojure, a Lisp dialect, so I type parens a lot).
Having this in the keyboard lets me get rid of some helper programs I had to install and configure on my computers (xcape on Linux and Karabiner on OS X -- both great utilities, but it is much nicer to just plug my keyboard in and get that behaviour).

The plugin now allows you to do the same thing with arbitrary keys (very much like another plugin which I'm not yet using, called DualUse), but this is the only thing I'm currently using it for.

### Other Fun Things

 - [TapDance][] lets me have a key that acts as Home when pressed once or End when double-tapped (also how my `[`/`]` key works)
 - [Unicode][] input in conjunction with [Macros][] lets me have some keys (on my `Aux` layer, in red on the above diagram) enter arbitrary Unicode characters -- I have Greek letters and a Thumbs-up emoji currently
 - [MagicCombo][] lets me define actions to happen when specific keys are held down -- I use this for switching to QWERTY (to let other folks try out the keyboard) by holding both palm keys and the top row of letters on both hands

## Software

Currently, the only way for people to modify their layout is to write an Arduino sketch in C++, compile it, and flash it to their device.
While the Keyboardio team and volunteers have done a lot to provide instructions to make this feasible for people, it is still not exactly optimal for non-programmers.
To this end, Gergely (a.k.a. algernon), who has written a massive amount of the Kaleidoscope firmware, started making [Chrysalis][].

However, it turns out having twins can cause a bit of a reduction in one's output, so Gergely is now *only* doing a giant amount of work on Kaleidoscope and a zillion plugins.
To keep Chrysalis going, Simon-Claudius stepped in to get it in working order.
While both of them are quite proficient in embedded systems sort of things, neither of them are experts in ClojureScript and the re-frame framework in which Chrysalis is written.
Luckily for me though, that is exactly what most of my work is in!

As such, I've been invited to join the team and help get Chrysalis in good working order -- just this morning, I was able to fix a vexing bug and got the keymap editor working!
Hopefully we'll get this out & available to users soon -- not only will this hopefully be very useful for the new Model 01 owners, [Dygma][] is planning on using Chrysalis as the configurator for their upcoming [Kaleidoscope-powered gaming keyboard][raise].
Look for more news on this front soon!

## Everything Else

Although the keyboard is lovely and I am very happy to have it, my favourite thing that has come out of this project is the delightful attitude Kaia and Jesse have fostered.
From their signoff being `<3` to their unrelentingly upbeat attitude in the face of constant frustrating setbacks, they have been a great force of positive energy, which has carried over to the community.
The Keyboardio [forums][forums] are full of friendly people offering help and advice to all the other folks there.
Contributing code and documentation to their Github is a great experience, with the maintainers (i.e. algernon and Jesse) making you feel welcome and appreciated.

Of all the things Jesse and Kaia have accomplished in making this project come into being, I think the thing they should be the most proud of is making such a kind and welcoming community.

❤️

  [arrived]: /2017/10/23/keyboard_arrived.html
  [backer_update]: https://www.kickstarter.com/projects/keyboardio/the-model-01-an-heirloom-grade-keyboard-for-seriou/posts/2029234
  [my_layout]: https://bitbucket.org/jamesnvc/keyboardiolayout/overview
  [OneShot]: https://github.com/keyboardio/Kaleidoscope-OneShot
  [SpaceCadet]: https://github.com/keyboardio/Kaleidoscope-SpaceCadet
  [TapDance]: https://github.com/keyboardio/Kaleidoscope-TapDance
  [Unicode]: https://github.com/keyboardio/Kaleidoscope-Unicode
  [Macros]: https://github.com/keyboardio/Kaleidoscope-Macros
  [MagicCombo]: https://github.com/keyboardio/Kaleidoscope-MagicCombo
  [Chrysalis]: https://github.com/Lepidopterarium/Chrysalis
  [Dygma]: http://www.dygma.com
  [raise]: http://www.dygma.com/dygma-raise/
  [forums]: http://community.keyboard.io/
