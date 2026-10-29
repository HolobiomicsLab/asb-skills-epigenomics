---
name: runtime-and-peak-memory-profiling
description: 'Use when comparing the computational efficiency of Hi-C compartment-calling
  methods: wall time and peak memory must be measured per tool across resolutions
  (10–100 kb) and sequencing depths on identical binned contact maps.'
when_to_use_negative:
- The goal is differential compartment detection or biological interpretation rather
  than benchmarking computational cost
- The input is not a Hi-C contact map/compartment-calling context (this profiling
  protocol is specific to genome-wide compartment callers)
- Only one tool is available — the skill requires head-to-head comparison on identical
  binned maps
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dcHiC
  role: 'Subject of the benchmark: performs compartment calling via parallelized partial
    SVD on a Filebacked Big Matrix (FBM); its wall time and peak memory are recorded
    with a process-level profiler'
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Comparator tool run on identical binned maps using HOMER Differential Scores,
    for runtime/memory comparison
- name: CscoreTool
  role: Comparator tool run on identical binned maps for runtime/memory comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:af7def32b8f700acadcc6505cf1de47765c8308d1ba294b6e9cd65e346e39684
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# Runtime and Peak Memory Profiling

## Summary

Benchmarks the runtime and peak memory of compartment-calling tools (dcHiC, HOMER, CscoreTool) on genome-wide Hi-C contact maps across resolutions and sequencing depths, quantifying the advantage of dcHiC's parallelized partial SVD on a Filebacked Big Matrix.

## When to use

Use when comparing the computational efficiency of Hi-C compartment-calling methods: wall time and peak memory must be measured per tool across resolutions (10–100 kb) and sequencing depths on identical binned contact maps.

## When NOT to use

- The goal is differential compartment detection or biological interpretation rather than benchmarking computational cost
- The input is not a Hi-C contact map/compartment-calling context (this profiling protocol is specific to genome-wide compartment callers)
- Only one tool is available — the skill requires head-to-head comparison on identical binned maps

## Inputs

- Genome-wide Hi-C contact maps (mouse ESC pseudo-replicates) binned at 10 kb, 25 kb, 50 kb, and 100 kb resolutions
- Down-sampled Hi-C datasets from serial down-sampling of a ~500M-read dataset at six depths

## Outputs

- Per-tool wall-time and peak-memory measurements at each resolution and sequencing depth
- Per-resolution and per-depth speedup ratios of dcHiC vs CscoreTool and dcHiC vs HOMER
- Peak-memory ranking of the three tools

## How to apply

Bin mouse ESC pseudo-replicate Hi-C contact maps genome-wide at 10 kb, 25 kb, 50 kb, and 100 kb resolutions, then run each compartment caller (dcHiC, HOMER, CscoreTool) on identical maps while recording wall time and peak memory with a process-level profiler. For depth scaling, repeat all three tools on the 50 kb maps at six serial down-sampling depths of the ~500M-read dataset. Compute per-resolution and per-depth speedup ratios of dcHiC against each comparator and rank tools by peak memory. Validate against the published ranges: dcHiC runs 4–13x faster than CscoreTool and 22–33x faster than HOMER at 100% sampling, and at 100 kb full depth the peak-memory ordering is CscoreTool (~0.24 Gb) < dcHiC (~0.34 Gb) < HOMER (~1.2 Gb). The speed advantage is mechanistically expected because dcHiC performs a parallelized partial SVD on a memory-efficient Filebacked Big Matrix (FBM), computing only the first few singular vectors needed for compartment analysis.

## Related tools

- **dcHiC** (Subject of the benchmark: performs compartment calling via parallelized partial SVD on a Filebacked Big Matrix (FBM); its wall time and peak memory are recorded with a process-level profiler) — https://github.com/ay-lab/dcHiC
- **HOMER** (Comparator tool run on identical binned maps using HOMER Differential Scores, for runtime/memory comparison)
- **CscoreTool** (Comparator tool run on identical binned maps for runtime/memory comparison)

## Evaluation signals

- dcHiC's speedup over CscoreTool falls within the reported 4–13x range at 100% sampling rate
- dcHiC's speedup over HOMER falls within the reported 22–33x range at 100% sampling rate
- At 100 kb full depth, peak-memory ordering is CscoreTool (~0.24 Gb) < dcHiC (~0.34 Gb) < HOMER (~1.2 Gb)
- All three tools were run on the identical binned maps at each of the four resolutions (10/25/50/100 kb) and six down-sampling depths
- Wall time and peak memory are captured by a process-level profiler, not self-reported timing

## Limitations

- Benchmark results are specific to mouse ESC pseudo-replicate Hi-C maps and the tested resolutions and depths; speedup ratios may not transfer to other organisms or data regimes
- dcHiC's efficiency claims rest on its partial SVD with FBM, so results do not generalize to tools without comparable memory-efficient data structures
- Compartment analysis from single-cell Hi-C remains limited by sparsity, which constrains benchmarking at very fine resolutions

## Evidence

- [intro] dcHiC implements a parallelized partial singular value decomposition using a memory-efficient Filebacked Big Matrix (FBM) data structure, computing only the first few singular vectors needed for compartment analysis: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [readme] README confirms dcHiC's optimized PCA speed as a headline feature supporting the benchmark rationale: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [intro] Article states the resolution range at which compartment analysis is feasible, bounding the benchmark's tested resolutions: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
