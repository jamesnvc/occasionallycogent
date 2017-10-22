---
layout: post
date: 2017-10-22
title: Keyboardio Cometh, Pt 4
---

*Part one of an n-part series. [Part 1](/2017/10/16/keyboardio_pt1.html), [Part 2](/2017/10/18/keyboardio_pt2.html), [Part 3](/2017/10/19/keyboardio_pt3.html)*

My Model 01 is scheduled to arrive tomorrow!
I am so excited to finally be able to use this thing.
I'm just crossing my fingers and hoping that nothing goes wrong...visions of delivery snafus or the errant shipping weight being something missing are haunting me.

In any case though, I continue to enjoy collaborating and contributing to Kaleidoscope & Chrysalis.
Being able to contribute documentation stuff to Kaleidoscope is very satisfying and is really helping me improve my understanding of how it all works.
I've been reading the source of the OneShot plugin as well as the base Kaleidoscope code and I think I'm almost confident that I know how things work...almost.

Chrysalis continues to be very enjoyable to work on.
Simon and Gergely [added me to the organization][org] & gave me commit rights, which I deeply appreciate as a measure of their confidence and trust.
Especially after so short a time collaborating, it was a gesture that meant a lot to me.

Implementing the keymap editor is proceeding well.
After setting up the infrastructure for tabs to group the different types of keys as Simon suggested, adding OneShot support was incredibly easy.
Things need a lot of cleanup, but it looks like it's really starting to come together!

{% include image.html url="https://s3.amazonaws.com/chat.leanpixel.com/uploads/59ed2f11-ef94-4a95-9672-42d68d63daac/screenshot-2017-10-22_19%3A51%3A20_EDT-04%3A00_1508716280.png" alt="Chrysalis keymap editor" caption="More progress on keymap editor - now supports OneShot keys!" %}

I think the next thing to work on is reviving the LED editor, as apparently it stopped working when some internals changed.
After that, who knows!
Lots of room for polish remains, but also maybe we can start thinking about other things that could be done with Chrysalis.
I like writing interpreters, so maybe we could add some ability to write simple functions that could be stored & interpreted from EEPROM?
That would enable use of things like macro and tap-dance keys from Chrysalis...now that would be a fun thing to make!

Fingers crossed that the next entry I write will be done on a Model 01!

  [org]: https://github.com/Lepidopterarium/
