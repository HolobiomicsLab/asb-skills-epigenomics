---
name: cross-tool-performance-comparison-analysis
description: Use when comparing the computational cost (wall time, peak memory) of
  alternative Hi-C compartment-calling tools on the same binned contact maps at multiple
  resolutions or down-sampling depths.
when_to_use_negative:
- The goal is accuracy or biological validation of differential compartments rather
  than runtime/memory profiling
- The analysis compares only a single tool against itself (no cross-tool baseline
  to benchmark against)
- Input is differential interaction calling or loop annotation, not compartment calling
  on binned contact maps
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dcHiC
  role: Subject of the benchmark; performs compartment calling via parallelized partial
    SVD on a Filebacked Big Matrix (FBM), expected 4-13x faster than CscoreTool and
    22-33x faster than HOMER at 100% sampling rate
  repo: https://github.com/ay-lab/dcHiC
- name: CscoreTool
  role: Comparator tool for genome-wide compartment calling; fastest-memory baseline
    (~0.24 Gb at 100 kb full depth) but 4-13x slower than dcHiC
- name: HOMER
  role: Comparator tool using HOMER Differential Scores; slowest and most memory-hungry
    (~1.2 Gb peak at 100 kb full depth), 22-33x slower than dcHiC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:19899bcf931266d0374528f149de9b42e740dceb01ce872c1b2a1d3dabd6c74f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - CscoreTool
  - HOMER
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-tool-performance-comparison-analysis

## Summary

Benchmark runtime and peak-memory footprints of compartment-calling tools (dcHiC, CscoreTool, HOMER) on identical binned Hi-C contact maps across resolutions and sequencing depths, quantifying per-tool speedup and memory scaling.

## When to use

Use when comparing the computational cost (wall time, peak memory) of alternative Hi-C compartment-calling tools on the same binned contact maps at multiple resolutions or down-sampling depths.

## When NOT to use

- The goal is accuracy or biological validation of differential compartments rather than runtime/memory profiling
- The analysis compares only a single tool against itself (no cross-tool baseline to benchmark against)
- Input is differential interaction calling or loop annotation, not compartment calling on binned contact maps

## Inputs

- Genome-wide binned Hi-C contact maps (mouse ESC pseudo-replicates) at 10 kb, 25 kb, 50 kb, and 100 kb resolutions
- Serial down-sampled versions of a ~500M-read 50 kb map (6 sequencing depths)
- Installed copies of dcHiC (R/Python 3+), HOMER, and CscoreTool

## Outputs

- Per-tool wall-time and peak-memory measurements at each resolution and sequencing depth
- Per-resolution and per-depth speedup ratios of dcHiC against CscoreTool and HOMER
- Peak-memory ranking of the three tools

## How to apply

Bin genome-wide Hi-C contact maps (e.g., mouse ESC pseudo-replicates) at several resolutions (10 kb, 25 kb, 50 kb, 100 kb) and run each compartment-calling tool on the identical binned maps. For dcHiC, exploit its parallelized partial singular value decomposition on a memory-efficient Filebacked Big Matrix (FBM), computing only the first few singular vectors needed for compartment analysis. Record wall time and peak memory per tool with a process-level profiler. To probe depth scaling, serially down-sample the ~500M-read 50 kb maps to 6 depths and rerun all tools, then compute per-resolution and per-depth speedup ratios and rank tools by peak memory.

## Related tools

- **dcHiC** (Subject of the benchmark; performs compartment calling via parallelized partial SVD on a Filebacked Big Matrix (FBM), expected 4-13x faster than CscoreTool and 22-33x faster than HOMER at 100% sampling rate) — https://github.com/ay-lab/dcHiC
- **CscoreTool** (Comparator tool for genome-wide compartment calling; fastest-memory baseline (~0.24 Gb at 100 kb full depth) but 4-13x slower than dcHiC)
- **HOMER** (Comparator tool using HOMER Differential Scores; slowest and most memory-hungry (~1.2 Gb peak at 100 kb full depth), 22-33x slower than dcHiC)

## Evaluation signals

- dcHiC's speedup over CscoreTool falls within the reported 4-13x range at 100% sampling rate
- dcHiC's speedup over HOMER falls within the reported 22-33x range at 100% sampling rate
- At 100 kb full depth, peak-memory ordering matches CscoreTool (~0.24 Gb) < dcHiC (~0.34 Gb) < HOMER (~1.2 Gb)
- All three tools receive byte-identical binned contact maps at each resolution/depth so measurements are attributable to the tools
- Profiler output captures process-level wall time and peak memory for every run (no missing samples across the 6 down-sampling depths)

## Limitations

- Benchmark results are specific to mouse ESC pseudo-replicate Hi-C maps and the tested resolutions/depths; other organisms or sparse single-cell maps (where compartment identification remains a major issue due to sparsity) may shift the ratios
- dcHiC's memory advantage depends on the FBM-backed partial SVD; gains at very coarse resolutions may be smaller than at 10-100 kb
- Speedup ratios are reported at 100% sampling rate and down-sampled 50 kb conditions only; intermediate configurations are not characterized

## Evidence

- [intro] dcHiC's parallelized partial SVD implementation is the mechanistic basis of its speed advantage: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [readme] The README advertises dcHiC's optimized PCA as faster than standard approaches, consistent with benchmark expectations: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [methods] HOMER's differential scoring is included in the paper as an explicit comparison point for compartment analysis: "HOMER Differential Scores (for comparison)"
