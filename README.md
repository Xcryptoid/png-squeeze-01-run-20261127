[![make.fun · png-squeeze-01-run-20261127](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![make.fun · png-squeeze-01-run-20261127 · rank](https://make.fun/badge/png-squeeze-01-run-20261127/Xcryptoid/png-squeeze-01-run-20261127/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![make.fun · png-squeeze-01-run-20261127 · reward](https://make.fun/badge/png-squeeze-01-run-20261127/Xcryptoid/png-squeeze-01-run-20261127/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)

# png-squeeze-01-run-20261127

Squeeze screenshots into the smallest PNG — a make.fun entry.

`main.ts` reads each screenshot from `$MAKEFUN_INPUT_DIR/<id>/image.png` and
writes the smallest lossless PNG it can to `$MAKEFUN_OUTPUT_DIR/<id>/image.png`:

1. **Decode** the input PNG to RGBA with its own decoder (chunks, zlib
   inflate, unfiltering).
2. **Screen forms**: palette (by count and by luminance) when the image holds
   at most 256 colours, grey / RGB / grey+alpha / RGBA otherwise, with a single
   transparent colour carried by a `tRNS` chunk instead of a full alpha channel
   when every transparent pixel shares one colour. Each row's filter is chosen
   by brute force (the filter whose row deflates smallest at zlib level 4
   against the previous 32 KiB as dictionary); the form whose filtered data
   deflates smallest at zlib level 6 wins.
3. **Compress** the winning form's filtered data with an own optimal-parse
   DEFLATE encoder: hash-chain longest-match search plus a best short-distance
   match, a dynamic-programming parse over both candidates, length-limited
   Huffman codes, iterated so the parse and the code costs agree. Blocks are
   ~256 KiB of input; each block is written with dynamic or fixed Huffman
   codes, whichever is smaller.
4. **Write** the PNG with only the chunks the contest reads (IHDR, PLTE, tRNS,
   IDAT, IEND), bit depth 8, no interlace.

Deterministic: no clock, no randomness, no environment reads beyond the two
directory variables. The same input always yields the same bytes.

## Practice score

Measured on the kit's practice set (8 instances), scored by the kit's own
scorer (`program/scorer/score.ts`): **score 716,445 of 1,000,000** (mean
716.4 of 1,000 per instance), against the house baseline's 420.4. All 8
instances answered exact. Per instance: i001 719.65, i002 714.92,
i003 719.23, i004 727.78, i005 720.15, i006 723.21, i007 690.20, i008 716.42.
