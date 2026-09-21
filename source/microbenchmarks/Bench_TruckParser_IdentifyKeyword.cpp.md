# source/microbenchmarks/Bench_TruckParser_IdentifyKeyword.cpp

> Compressed twin: a benchmark comparing ways to recognise the section keyword at the start of a truck-file line.

**Needs** — the Google Benchmark library (build-time only; not part of the game)
**Used by** — nothing (stand-alone executable)
**Tier floor** — T2

## Purpose

Exploration behind a design question in [`resources/rig_def_fileformat`](../main/resources/rig_def_fileformat/README.md): how the truck-file parser should classify each line. It embeds a real ~1400-line truck file (a 2012 school bus) and the keyword list, and times each strategy over every line. Self-contained; a rebuild needs it only if it wants to re-check the decision.

## Asserted behaviours / measured variants

- **Keyword set** — the benchmark's keyword enum lists the truck-file keywords recognised at the time (add_animation … wings, 105 keywords; unknown = invalid). A keyword matches only as the first token of a line, case-insensitively; block keywords may stand alone on their line.
- **sol1 — one big case-insensitive regular expression** with one optional capture group per keyword; the matching group gives the keyword. Variants add a cheap precondition that skips the regex when the first character is not a letter (b), not a digit (c), or is alphabetic (d).
- **sol2 — switch on the first character** (either case), then compare the rest of the line case-insensitively against the few keywords starting with that letter. The comparison is whole-line, so as written it only recognises lines consisting of the keyword alone (the embedded sample lines carry trailing blanks and arguments, so most return "invalid") — the timing is still indicative of the approach. Variant b adds the "first character must be a letter" precondition.
- The benchmark only times; it does not check that the strategies agree, and it records no result numbers.

## Notes

The README in that directory states the convention: each file is a self-contained micro-benchmark added when a change's performance impact is uncertain.
