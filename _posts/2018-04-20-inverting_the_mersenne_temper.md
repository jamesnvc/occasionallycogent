---
title: How to Invert the Mersenne Twister's Temper (If You're Bad at Bit Twiddling)
layout: post
date: 2018-04-20
---

**Warning: Minor spoilers for Cryptopals challenge 23 here. Don't read if you haven't completed it yet!**

I've recently being trying my hand at [the Cryptopals Challenges][cryptopals], implementing my solutions in Prolog ([my new favourite language][occ_prolog])

Challenge 23 involves inverting the "temper" operation of the [Mersenne Twister random number generator][mersenne].
While I'm sure this is easy to people that live & breathe bitwise operations and such, understanding how to do this required a fair amount of thinking on my part, so I thought I'd go ahead and document my journey to understanding.

I won't belabour the details of how the Mersenne Twister works in general (the Wikipedia page linked above has a sufficient explanation of the algorithm), but I'll just point on the temper operation in particular.

This is the step after we get the next in our sequence of random numbers, wherein, to quote Wikipedia:

> the Mersenne Twister is cascaded with a tempering transform to compensate for the reduced dimensionality of equidistribution

I don't quite have the math to know precisely what that means (while I'm casually trying to pick up some more abstract math, I just have an undergrad engineering degree, so my experience is mainly in more applied areas), but luckily we don't really need to understand what it's doing to implement it.
The code for the tempering transform looks like this (in Prolog, but it should be fairly straightforward what it's doing):

```prolog
%! temper(+X, -Z) is det.
%
% True when Z is the tempered transform of X.
temper(X, Z) :-
    % MT19937 parameters
    U = 11, D = 0xffff_ffff,
    S = 7,  B = 0x9d2c5680,
    T = 15, C = 0xefc60000,
    L = 18,

    Y1 is X xor ((X >> U) /\ D),
    Y2 is Y1 xor ((Y1 << S) /\ B),
    Y3 is Y2 xor ((Y2 << T) /\ C),
    Z is Y3 xor (Y3 >> L).
```

The constants are defined as part of the particular algorithm implementation (in this case MT19937, i.e. based on the Mersenne prime 2<sup>19937</sup>-1), the `/\` operator is bitwise-and, and the four xor/shift/and operations are the tempering transform.

So, how do we invert this?

I was trying to deduce this myself, but I got started with what I realize in hindsight was a mistaken assumption, and got quite frustrated when my code would seem to randomly oscillate between working and not working in a way that was absolutely baffling to me.
Because this is just a fun side project and I didn't feel like banging my head against it forever, I did a search to try to find some guidance on what I was missing and found [this rather taciturn solution][untemper].
The implementation of the above algorithm in Prolog looks like this:

```prolog
%! untemper(+Z, -X) is det.
%
% True when Z is the tempered transform of X.
untemper(Z, X) :-
    U = 11, %D = 0xffff_ffff,
    S = 7,  %B = 0x9d2c_5680,
    T = 15, C = 0xefc6_0000,
    L = 18,
    Y1 is Z xor (Z >> L),
    Y2 is Y1 xor ((Y1 << T) /\ C),
    Y3 is Y2 xor ((Y2 << S) /\ 0x0000_1680), % 13 or 14 bits
    Y4 is Y3 xor ((Y3 << S) /\ 0x000c_4000), % 6 or 7 bits
    Y5 is Y4 xor ((Y4 << S) /\ 0x0d20_0000), % 8 bits
    Y6 is Y5 xor ((Y5 << S) /\ 0x9000_0000), % 4 bits
    Y7 is Y6 xor ((Y6 >> U) /\ 0xffc0_0000), % 10 bits
    Y8 is Y7 xor ((Y7 >> U) /\ 0x003f_f800), % 11 bits
    X  is Y8 xor ((Y8 >> U) /\ 0x0000_07ff). % 11 bits
```

While the solution worked, I didn't understand *why* it worked and was a little uncomfortable moving forwards in ignorance.
While the simple relation for the first transformation (`Z = Y3 xor (Y3 >> L)` ⟷ `Y3 = Z xor (Z >>L)`) is nice, why does it work?
And why do the other steps require being broken down in to multiple sub-steps?

Let's start with how the first untemper step for Z ⟷ Y<sub>3</sub> works and make a little diagram to examine the bit pattern.


           ↙ bit w          ↙ bit w-L
           YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY    Y3
         ⊻ 000000000000000000YYYYYYYYYYYYYY    Y3 >> L
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ    Z
           ╰─────L bits─────╯╰──W-L bits──╯
                  = 18 bits        = 14 bits
        W = 32 (32 bit word size)
        L = 18

So, we can see that the L high bits of Z are just the L high bits of Y3, because they're xor'd against zero.
The W-L low bits of Z are the W-L high bits of Y3 xor'd against the W-L low bits.
We now know the L high bits of Y3 are the high bits of Z and because L > (W - L), we can xor the W - L high bits of Z against the W - L low bits of Z to yield the W - L low bits of Y3.

Putting it together, this gives us `Y3 = Z ⊻ (Z >> L)`.


The next step (Y<sub>3</sub> ⟷ Y<sub>2</sub>) looks like this:

           ↙ bit w         ↙ bit T
           YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY    Y2
       ⊻ ( YYYYYYYYYYYYYYYY0000000000000000    (Y2 << T)
         ∧ CCCCCCCCCCCCCCCC0000000000000000 )  C
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ    Y3
           ╰────W-T bits──╯╰──────T───────╯
                = 17 bits         = 15 bits
        W = 32 (32 bit word size)
        T = 15

It is essentially the same, except we have the additional AND step, but the shifting-down works the same[^waitwhat].

What about `Y2 = Y1 xor ((Y1 << S) /\ B)`?
(In the diagram below, I'm using Z to represent the bits of Y2)


           ↙ bit w                  ↙ bit S
           YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY    Y1
       ⊻ ( YYYYYYYYYYYYYYYYYYYYYYYYY0000000    (Y2 << S)
         ∧ BBBBBBBBBBBBBBBBBBBBBBBBB0000000 )  B
         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ    Y2
           ╰───────W-S bits────────╯╰──S──╯
                   = 14 bits          = 7 bits
        W = 32 (32 bit word size)
        S = 7

How do we invert this?

As previously, we trivially get that the low S=7 bits of Y2 are the low S bits of Y1, since it was xor'd against zero.
However, we now we only have 7 bits known and need to recover W-S = 25 bits...

What we have to do then, is recover Y1 in chunks of S bits at a time:
If we mask out only the next S bits, we can recover those in the same manner as before:

           ↙ bit w     bit 2S↘      ↙ bit S
           ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ    Y2
       ⊻ ( ZZZZZZZZZZZZZZZZZZZZZZZZZ0000000    (Y2 << S)
         ∧ 000000000000000000BBBBBBB0000000 )  Mask /\ B
       ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           ZZZZZZZZZZZZZZZZZZYYYYYYYYYYYYYY
           ╰───W-2S bits────╯╰──S──╯╰──S──╯
                   = 14 bits          = 7 bits
        W = 32 (32 bit word size)
        S = 7

Now, the high W - 2S bits of Y2 are unchanged, the low S bits remain the low S bits of Y1, and now the second-lowest S bits of Y2 are xor'd against the S low bits of Y2 /\ B which, since the S low bits of Y2 are the S low bits of Y1, gives us the next S bits of Y1!

After this operation, we have the low 2S bits of Y1 and the high W-2S bits of Y2. We can repeat the operation, shifting the mask down by a further S bits to get the low 3S bits of Y1 and continue until we have the whole thing.

Using the same method, we can invert the last step to get X from Y1, albeit going in the other direction (because it does a right shift) and in fewer steps (because it shifts by 11 bits and a time, instead of 7).

Here's the Prolog code for the invert step, in a slightly different way, so the relation between the parameters and the constants being and-ed against is more clear:

```prolog
%! untemper(+Z, -X) is det.
%
% True when Z is the tempered transform of X.
untemper(Z, X) :-
    U = 11, %D = 0xffff_ffff,
    S = 7,  B = 0x9d2c_5680,
    T = 15, C = 0xefc6_0000,
    L = 18,
    Y1 is Z xor (Z >> L),
    Y2 is Y1 xor ((Y1 << T) /\ C),
    SMask is (1 << S) - 1,
    Y3 is Y2 xor ((Y2 << S) /\ B /\ (SMask << S)),
    Y4 is Y3 xor ((Y3 << S) /\ B /\ (SMask << (S*2))),
    Y5 is Y4 xor ((Y4 << S) /\ B /\ (SMask << (S*3))),
    Y6 is Y5 xor ((Y5 << S) /\ B /\ (SMask << (S*4))),
    UMask is (1 << U) - 1,
    % UMask is and'd with D, but that's all ones, so we can just use
    % the mask by itself
    Y7 is Y6 xor ((Y6 >> U) /\ (UMask << (U*2))),
    Y8 is Y7 xor ((Y7 >> U) /\ (UMask << U)),
    X  is Y8 xor ((Y8 >> U) /\ UMask).
```

I'm pretty sure that the intersection of the set of people that are interested in inverting the Mersenne temper step and the set of people for whom doing this sort of bit manipulation is absolutely trivial is the empty set, but writing this definitely helped me improve my understanding.

  [cryptopals]: https://www.cryptopals.com
  [occ_prolog]: /2018/03/24/discovering_an_old_language.html
  [mersenne]: https://en.wikipedia.org/wiki/Mersenne_Twister
  [untemper]: https://randombit.net/bitbashing/2009/07/21/inverting_mt19937_tempering.html

  [^waitwhat]: I added this diagram at the end of writing this post and realized that this seems to not account for the two highest bits? I'm now not sure why this step works...
