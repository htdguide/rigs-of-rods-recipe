# source/microbenchmarks

> Stand-alone performance experiments; not part of the game build.

Each file is a self-contained benchmark written when the performance impact of a change was uncertain. There is currently one: [`Bench_TruckParser_IdentifyKeyword.cpp`](Bench_TruckParser_IdentifyKeyword.cpp.md), which compares a first-character switch against one large regular expression for classifying truck-file lines. A rebuild may ignore this chapter.
