# source/main/gameplay/RandomTreeLoader.h

> An unused example tree-page loader for the vegetation-paging library that scatters random trees into empty pages.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — nothing — unused in this revision
**Tier floor** — T2

## Purpose

Not included anywhere in the program: dead code kept from the vegetation library's samples. A rebuild may omit it. Recorded for completeness.

## State

Stateless beyond the paging library's page grids.

## `RandomTreeLoader`

**Contract** — on page load, if the page has no trees, add `page_size/10` trees at random positions, yaw and scale 0.07–0.13 (packed into 16-bit position and 8-bit rotation/scale), then load normally; on unload, remove all trees of the page.
