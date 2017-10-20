---
layout: post
date: 2017-10-19
title: Keyboardio Cometh, Pt 3
---

*Part one of an n-part series. [Part 1](/2017/10/16/keyboardio_pt1.html), [Part 2](/2017/10/18/keyboardio_pt2.html)*

Model 01 status:
Arrived in Perrysburg, Ohio; still on track to receive it Monday.

The day started with a very nice email from the CTO of a company called [Dygma][] that is making a keyboard (possibly two? I think they have the extant [Shortcut][] and the new [Raise][]) that will be using Kaleidoscope and wants to have a nice graphical configuration tool in Chrysalis.
They said that they had seen the things I had started pushing on the repo, had heard from Simon that I'd be helping out on the project, and wanted to let me know that my work is appreciated.

It was a very nice way to start the day and really emphasized how I ended the last post about working on Chrysalis & Kaleidoscope, about how rewarding it can be to do work that others appreciate.

I also got some slightly alarming news about potential issues with my in-flight Model 01, but that will hopefully turn out to be just a little bit of measurement error...fingers crossed.
It is pretty cool that one of the two founders of Keyboardio personally reached out to me after noticing a tiny inconsistency though -- especially when the co-founders are also new parents!

The front-end part of the Chrysalis work is proceeding well.
I had a chat with Simon about how we want the keyboard layout editor to work, which I think is going to be pretty nice.

{% include image.html url="https://s3.amazonaws.com/chat.leanpixel.com/uploads/59e93f43-2f6a-44a7-a468-865fa0de87e3/screenshot-2017-10-19_20%3A11%3A34_EDT-04%3A00_1508458294.png" alt="Chrysalis keymap editing" caption="Keymap editor in progress - hopefully I'll have a real device to test it on soon!" %}

I also spent some time today trying to learn how Chrysalis will communicate with the actual device.
The general idea is that the keyboard can listen on the serial port and various plugins (in this case, the [EEPROM-Keymapper plugin][keymapper]) can register a [Focus][] hook to be called when it recieves some data with a particular prefix.
There is already code that can send the appropriate prefix to set key mappings over the serial port, but we need to add some code to translate the ClojureScript maps representing keys (e.g. `{:plugin :core, :key :a, :modifiers #{:shift}}`) into a representation that Kaleidoscope can understand (spolier: the above would be (I think) `0x0804`)

Figuring out how to do this mapping was fairly straightforward, but took me on a fun tangent.
Reading the source of [`key_defs.h`][key_defs] seemed to indicate that the key was represented as a sixteen-bit number, where the high bits are the flags and the low bits are the keyCode; like so:

    // in Kaleidoscope/src/key_defs.h
    typedef union Key_ {
        // ATMega32U4 is little-endian, so keyCode is the low bits
        struct {
            uint8_t keyCode;
            uint8_t flags;
        };
        uint16_t raw;
        // bunch of operator overloads elided...
    } Key;

Asking some questions from the very helpful Algernon on IRC, though, indicated that this was an incomplete view of the picture.
While for normal keyboard keys, the low eight bits give the keycode of the pressed key and flags are used to indicate modifiers held, there are other uses.
In particular, plugins often will want to have their own "special" keys that won't be interpreted as normal keys.
So, for example, when you put a macro key in your key map, the `Key` that is sent can be distinguished as something that the Macro plugin should handle, and not be treated as a normal key, or sent to another plugin.

This is done by setting the high bit of `flags` (which is the [`RESERVED` bitmask in `key_defs.h`][keydef_bitmasks]).
When this flag is set, it indicates that this `Key` should no longer be treated like two 8-bit values as normal, but can have any meaning that the plugin which created it cares to assign.

You may wonder though, how to plugins avoid creating `Key`s that conflict with each other?
We know that setting the reserved bit means that it won't be confused for an ordinary key press, but how do we ensure that a macro key isn't misinterpreted as, say, a one-shot key?

The solution lies in a library that had up until now seemed completely arcane to me, [Kaleidoscope-Ranges][].
This single-file plugin defines one enum, is included by most plugins, but provided no indication of what the enum meant.
After learning how plugins need to inject non-conflicting key codes though, this finally made sense.
The enum is used to define the range of raw, sixteen-bit values that each plugin will inject!

The enum starts with `FIRST = 0xc000`, which is `11000000 00000000` in binary -- that is, flag bit `RESERVED` <strike>and `INJECTED` set, so the minimum value that could be injected by a plugin</strike>, plus another bit to reserve space for plugins operating without `kaleidoscope::ranges`.[^correction]
From there we see the various plugins defining the ranges that they will use (e.g. the first is the OneShot plugin, from `OS_FIRST` to `OS_LAST`, plus some internal separation into one-shot modifiers and one-shot layers).

I was very happy to have figured this out and wanted to make sure that, in the future, people could understand this without needing to read a bunch of source from other libraries & bug Algernon and Jesse on IRC, so I tried to write up what I had learned:
Behold, a [pull request][ranges_pr] that makes the [README][] almost twice as long as the code! 😛

All in all, a productive & educational day.

  [Dygma]: http://www.dygma.com/
  [Shortcut]: http://shortcut.gg
  [Raise]: http://www.dygma.com/dygma-raise/

  [keymapper]: https://github.com/keyboardio/Kaleidoscope-EEPROM-Keymap
  [Focus]: https://github.com/keyboardio/Kaleidoscope-Focus
  [key_defs]: https://github.com/keyboardio/Kaleidoscope/blob/8741c68f7b378a8c3efb152b2a828ca0f4a30836/src/key_defs.h#L20
  [keydef_bitmasks]: https://github.com/keyboardio/Kaleidoscope/blob/8741c68f7b378a8c3efb152b2a828ca0f4a30836/src/key_defs.h#L76

  [Kaleidoscope-Ranges]: https://github.com/keyboardio/Kaleidoscope-Ranges/blob/master/src/Kaleidoscope-Ranges.h
  [ranges_pr]: https://github.com/keyboardio/Kaleidoscope-Ranges/pull/2
  [README]: https://github.com/keyboardio/Kaleidoscope-Ranges

  [^correction]: Updated 20 October 2017: Thanks [Algernon for correcting my confusion](https://trunk.mad-scientist.club/@algernon/98860220164360732) about how `INJECTED` works -- that flag is for keyState, not the `Key`, to prevent infinite loops of plugins changing key codes and then changing the key that they just injected.
