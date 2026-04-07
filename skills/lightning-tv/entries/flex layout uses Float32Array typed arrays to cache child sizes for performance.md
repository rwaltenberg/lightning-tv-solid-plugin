---
description: The flex engine pre-allocates Float32Array buffers for child dimensions and margins to avoid GC pressure during layout passes in long-running TV applications
type: architecture
module: core
created: 2026-04-07
---

# flex layout uses Float32Array typed arrays to cache child sizes for performance

Both `flex.ts` and `flexLayout.ts` use `Float32Array` instead of regular JavaScript arrays to store intermediate child size calculations.

## Allocated buffers

For each layout pass, 7 typed arrays are allocated with length equal to `numProcessedChildren`:
- `childMainSizes` — base sizes along main axis
- `childMarginStarts` — leading margins
- `childMarginEnds` — trailing margins
- `childTotalMainSizes` — mainSize + marginStart + marginEnd
- `childCrossSizes` — sizes along cross axis
- `childMarginCrossStarts` — cross-axis leading margins
- `childMarginCrossEnds` — cross-axis trailing margins

## Why typed arrays

`Float32Array` is a fixed-size buffer with 32-bit float entries. Compared to regular arrays:
- No boxing overhead per element
- Contiguous memory layout — better cache locality during sequential reads
- No GC pressure for element values (buffer itself is one GC object)

TV applications often have large grids with hundreds of visible flex children. Layout runs on every content update. The typed array approach avoids creating thousands of temporary number objects per frame.

## Trade-off

`Float32Array` has 32-bit precision (vs 64-bit for JavaScript numbers). For pixel coordinates this is sufficient — 32-bit float represents values up to ~16 million exactly, and sub-pixel differences below ~0.01 are imperceptible.

---

Related Entries:
- [[flex layout engine positions children using justifyContent along the main axis]] — these arrays feed the positioning loops
- [[flexGrow distributes positive available space proportionally among children]] — growths are applied by mutating these arrays
- [[Row component is a horizontal navigable list with built-in flex layout and 30px gap]] — Row is the primary flex container in TV apps; this optimization is critical when Row contains many card items in a grid
- [[VirtualGrid slices items by row boundaries and uses flexWrap for 2D layout]] — VirtualGrid's layout pass benefits from typed arrays when re-slicing causes a full flex recalculation with many items

Source: [[core-flex]], [[core-flexLayout]]
Domains:
- [[styling guide]], [[performance guide]]
