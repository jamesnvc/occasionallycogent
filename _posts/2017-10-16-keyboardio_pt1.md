---
layout: post
date: 2017-10-16
title: Keyboardio Cometh, Pt 1
tags: keyboardio
---

*Part one of an n-part series. [Part 2](/2017/10/18/keyboardio_pt2.html), [Part 3](/2017/10/19/keyboardio_pt3.html)*

I've been having a lot of fun today doing some contribution to the Keyboardio community.
Now that people are starting to get their keyboards, new folks are coming on to the forums and asking questions that I can actually answer, despite not having a keyboard.
It was immensely satisfying to be able to give someone some code for a macro they wanted and have it work without me ever having put code on an actual device!

I've also been doing some more contribution to the plugin stuff.
I had done some copy-editing passes on some of algernon's documentation before, but I'm starting to get to slightly larger & more interesting things.
I made a diagram to explain the fairly arcane `RxCy` key coordinates that users only have to deal with in [one particular plugin][magiccombo].
It was a bit tedious to clean up the SVGs that the Keyboardio folks have provided for keycap layout, but it looks much nicer than using one of the cruder keyboard layout tools!

Having some of the fire for using the keyboard stirred in me again, I also did some work on my own custom layout.
Something I thought would be neat was adding a Greek letter numpad on the left hand on the layer that has the numpad on the right (I like to use Greek letters for variable names in mathy code).
The implementation of this was fairly straightforward, but after adding it, I started getting a warning when compiling the sketch that I was running out of dynamic memory (i.e. SRAM, as opposed to program memory or EEPROM).
Algernon was very helpful on the forums and, although I was initially confused as to what on earth was taking up so much memory, as my sketch is all enums, functions, and PROGMEM, I eventually realized some of the plugins I was using were talking up a ton of memory.

The [LED-Stalker][stalker] plugin was my first guess as to where the hogs were, as I had seen that algernon had conditionally commented his out and I figured there was a reason for that.
Removing that plugin dropped memory usage by about 4% (nothing to sneeze at) and looking at the code, it was clear why:
While most other LED effect plugins are fairly simple and don't use much memory, since this plugin needs to keep a history of movement, it has a big 2D array (which, when the device only has 2560 bytes of RAM, eats up a lot).

I wasn't sure where else the memory usage was coming from, but Algernon tipped me off to the fact that the HostOS checking functionality consumes a lot of RAM.
This was easy to verify by setting `KALEIDOSCOPE_HOSTOS_GUESSER` to 0 and seeing that the amount of RAM consumed dropped by 10%.
While I can manually set the HostOS, I would like to be able to use this keyboard on both my OS X laptop and my Linux desktop.
I looked at the source for the HostOS Guesser, but that just wrapped the FingerprintUSBHost library.
I looked at the FingerprintUSBHost library and saw that it had a giant array of `USBSetup` structs that it wrote to but never read!

I removed it and submitted a PR; Jesse apparently uses it for debugging, so I re-added it, wrapped in an `#ifdef DEBUG`.
Hopefully I'll get my [first code PR][hostos_pr] for Kaleidoscope landed soon!

The advantage of not getting my keyboard right away:
Bugs get fixed and features get added in the firmware before I start using it!

  [stalker]: https://github.com/keyboardio/Kaleidoscope-LED-Stalker
  [magiccombo]: https://github.com/keyboardio/Kaleidoscope-MagicCombo
  [hostos_pr]: https://github.com/keyboardio/FingerprintUSBHost/pull/4
