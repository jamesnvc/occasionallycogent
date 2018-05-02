---
title: SHA-1 Hash in Pure Prolog
layout: post
data: 2018-05-01
---

I'm continuing to work on [the cryptopals challenges](https://cryptopals.com) and it remains very fun.

The [challenge I just finished](https://www.cryptopals.com/sets/4/challenges/28) asks you to find an implementation of the SHA-1 algorithm in the language you're using.
I wasn't able to find an implemention of the algorithm in pure Prolog (the [SWI-Prolog implementation](http://www.swi-prolog.org/pldoc/doc/_SWI_/library/sha.pl?show=src) uses a foreign library to provide the function), so I went ahead & did it myself.

I wanted to include it here, just in case someone else may find it useful:

(note that, of course, SHA-1 isn't really safe to use -- this is just for fun).

```prolog
:- module(s4c28, [sha1/2]).

:- use_module(library(apply), [foldl/4]).
:- use_module(library(clpfd)).
:- use_module(library(list_util), [take/3, drop/3, replicate/3]).

blocks([], _, []) :- !.
blocks(Bytes, BSize, [Block|NextBlocks]) :-
    take(BSize, 0, Bytes, Block),
    drop(BSize, Bytes, NextBytes),
    blocks(NextBytes, BSize, NextBlocks).

int_bytes_be(I, Bs) :-
    foldl([E, I0, I]>>(
              E in 0..255,
              I #= I0*256 + E),
         Bs, 0, I).

int_bytes_be_64(I, B) :-
    int_bytes_be(I, B),
    length(B, 8), !.

int_bytes_be_32(I, B) :-
    int_bytes_be(I, B),
    length(B, 4), !.

sha1(In, Digest) :-
    In ins 0..255,

    sha1_preprocess(In, Message),

    % process the message in successive 512-bit (64-byte) chunks
    blocks(Message, 64, MsgBlocks),

    % process chunks
    H0 = 0x67452301,
    H1 = 0xefcdab89,
    H2 = 0x98badcfe,
    H3 = 0x10325476,
    H4 = 0xc3d2e1f0,

    foldl([Block, H0-H1-H2-H3-H4, G0-G1-G2-G3-G4]>>(
              sha1_process_chunk(H0-H1-H2-H3-H4, Block, A-B-C-D-E),
              G0 #= (H0 + A) /\ 0xffff_ffff,
              G1 #= (H1 + B) /\ 0xffff_ffff,
              G2 #= (H2 + C) /\ 0xffff_ffff,
              G3 #= (H3 + D) /\ 0xffff_ffff,
              G4 #= (H4 + E) /\ 0xffff_ffff
          ), MsgBlocks, H0-H1-H2-H3-H4, Hh0-Hh1-Hh2-Hh3-Hh4),

    Out #= ((Hh0 << 128) \/ (Hh1 << 96) \/ (Hh2 << 64) \/ (Hh3 << 32) \/ Hh4),
    int_bytes_be(Out, Digest), length(Digest, 20), !.

sha1_preprocess(Input, Message) :-
    length(Input, InL),
    Ml #= InL * 8, % original message length in bits

    % append the bit '1' to the message
    append(Input, [0x80], M1),
    % append zeros to the message st resultant length in bits is
    % congruent to −64 ≡ 448 (mod 512)
    % 64 bits = 8 bytes, 512 bits = 64 bytes
    % -8 ≡ 56 (mod 64)
    K #= (56 - (InL + 1)) mod 64,
    replicate(K, 0, Zeroes),
    append(M1, Zeroes, M2),

    % append the original length as a big-endian 64-bit integer
    int_bytes_be_64(Ml, MlBytes),
    append(M2, MlBytes, Message),

    length(Message, MessageLen),
    0 #= MessageLen mod 64.

leftrotate(I, N, RotI) :-
    WordLen #= 32,
    WordMask #= (1 << WordLen) - 1,
    RotI #= ((I << N) /\ WordMask) \/ (I >> (WordLen - N)).

sha1_process_chunk(H0-H1-H2-H3-H4, Chunk, ChunkHash) :-
    % break each chunk into 16 32-bit (4-byte) big-endian words
    blocks(Chunk, 4, Words_),
    length(Words_, 16),
    maplist(int_bytes_be_32, Words, Words_),

    % extend the 16 32-bit words into 80 32-bit words
    Extra #= 80 - 16,
    length(ExtraWords, Extra),
    append(Words, ExtraWords, ExtendedWords),
    numlist(16, 79, Indices),
    maplist({ExtendedWords}/[I,W]>>(
                I3 is I - 3, I8 is I - 8,
                I14 is I - 14, I16 is I - 16,
                nth0(I3, ExtendedWords, W3),
                nth0(I8, ExtendedWords, W8),
                nth0(I14, ExtendedWords, W14),
                nth0(I16, ExtendedWords, W16),
                W_ is W3 xor W8 xor W14 xor W16,
                leftrotate(W_, 1, W)
            ), Indices, ExtraWords),

    % main loop
    numlist(0, 79, Is),
    foldl({ExtendedWords}/[I, A0-B0-C0-D0-E0, A-B-C-D-E]>>(
              main_loop_step(I, B0-C0-D0, F, K),
              E = D0,
              D = C0,
              leftrotate(B0, 30, C),
              B = A0,
              leftrotate(A0, 5, X),
              nth0(I, ExtendedWords, Wi),
              A #= (X + F + E0 + K + Wi) /\ 0xffff_ffff
          ), Is, H0-H1-H2-H3-H4, ChunkHash).

main_loop_step(I, B-C-D, F, K) :-
    between(0, 19, I),
    F #= (B /\ C) \/ ((\B) /\ D),
    K = 0x5a827999.
main_loop_step(I, B-C-D, F, K) :-
    between(20, 39, I),
    F #= B xor C xor D,
    K = 0x6ed9eba1.
main_loop_step(I, B-C-D, F, K) :-
    between(40, 59, I),
    F #= (B /\ C) \/ (B /\ D) \/ (C /\ D),
    K = 0x8f1bbcdc.
main_loop_step(I, B-C-D, F, K) :-
    between(60, 79, I),
    F #= B xor C xor D,
    K = 0xca62c1d6.

% tests
:- use_module(library(crypto), [hex_bytes/2]).
:- use_module(library(plunit)).

:- begin_tests(s4c28).

test(sha1_empty) :-
    sha1(``, O),
    hex_bytes(Hex, O), !,
    Hex = 'da39a3ee5e6b4b0d3255bfef95601890afd80709'.

test(sha1_dog) :-
    sha1(`The quick brown fox jumps over the lazy dog`, O),
    hex_bytes(Hex, O),
    Hex = '2fd4e1c67a2d28fced849ee1bb76e7391b93eb12'.

test(sha1_cog) :-
    sha1(`The quick brown fox jumps over the lazy cog`, O),
    hex_bytes(Hex, O),
    Hex = 'de9f2c7fd25e1b3afad3e85a0bd17d9b100db4b3'.

test(sha1_do) :-
    sha1(`The quick brown fox jumps over the lazy do`, O),
    hex_bytes(Hex, O),
    Hex = 'f2f38c31109f8a9421f301e9d554193bc4274a32'.

:- end_tests(s4c28).
```

Hope this may prove useful, or at least interesting, to you!
