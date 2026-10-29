---
name: gsc-permutation-shuffling-benchmark
description: Use when a genomic overlap/enrichment analysis requires genome-scale
  null range sets and runtime is a bottleneck, or when benchmarking shuffling (GSC)
  versus block bootstrap (bootRanges) approaches on ChIP-seq peak sets.
when_to_use_negative:
- 'The goal is reliability of null distributions rather than speed: the paper shows
  shuffling or permutation schemes may result in overly narrow test statistic null
  distributions and over-estimation of statistical significance, so benchmarking runtime
  alone does not justify choosing shuffling.'
- The analysis target is small, non-genome-scale range sets where runtime is not a
  limiting factor and GSC timing differences are immaterial.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Block bootstraps genomic ranges to generate null range sets whose generation
    time (0.30 s without, 0.37 s with overlap computation) is measured against GSC
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges, used to produce the block bootstrapped
    null ranges in the timing comparison
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Permutation/shuffling-based method timed on identical peak sets and hardware
    as the runtime baseline against bootRanges
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:670f4575a15b6ebfe9ac44415f12a8ad4e41e7ddcee9df6ea21921885bada09a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gsc-permutation-shuffling-benchmark

## Summary

A timing benchmark protocol that compares GSC permutation-based shuffling against bootRanges block bootstrapping of null genomic range sets, demonstrating that bootRanges generates null ranges roughly 20-fold faster (mean 0.30 s per genome bootstrap vs 7.56 s for GSC) on the same hardware.

## When to use

Use when a genomic overlap/enrichment analysis requires genome-scale null range sets and runtime is a bottleneck, or when benchmarking shuffling (GSC) versus block bootstrap (bootRanges) approaches on ChIP-seq peak sets.

## When NOT to use

- The goal is reliability of null distributions rather than speed: the paper shows shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so benchmarking runtime alone does not justify choosing shuffling.
- The analysis target is small, non-genome-scale range sets where runtime is not a limiting factor and GSC timing differences are immaterial.

## Inputs

- ENCODE kidney ChIP-seq peak set as GRanges
- ENCODE bladder ChIP-seq peak set as GRanges
- Segmentation-derived block lengths for segmental block bootstrap construction
- Target feature GRanges for the downstream overlap computation

## Outputs

- timing_benchmark.csv with mean generation times for bootRanges (with and without overlap computation) and GSC
- Speedup factor (roughly 20-fold for bootRanges on the same hardware)

## How to apply

Load the ENCODE kidney and bladder ChIP-seq peak sets and construct GRanges objects with the GRanges_construct.R preprocessing script. Time bootRanges block-bootstrap generation of null range sets on each peak set using segmental blocks matched to the segmentation-derived block lengths, then repeat the timing including downstream overlap computation of bootstrapped ranges against target features (the 'with overlaps' timing). On identical peak sets and hardware, time GSC permutation-based shuffling to obtain its mean runtime. Tabulate mean generation times side by side (bootRanges ~0.30 s, ~0.37 s with overlaps, vs GSC 7.56 s), compute the speedup factor, and write results to timing_benchmark.csv.

## Related tools

- **bootRanges** (Block bootstraps genomic ranges to generate null range sets whose generation time (0.30 s without, 0.37 s with overlap computation) is measured against GSC) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges, used to produce the block bootstrapped null ranges in the timing comparison) — https://bioconductor.org/packages/nullranges
- **GSC** (Permutation/shuffling-based method timed on identical peak sets and hardware as the runtime baseline against bootRanges)

## Examples

```
Rscript script/bootranges_vs_GSC.R
```

## Evaluation signals

- Recorded mean bootRanges block bootstrap time should be approximately 0.30 s per genome bootstrap (0.37 s when the overlap computation is included)
- Recorded mean GSC runtime on the identical peak sets and hardware should be approximately 7.56 s
- The computed speedup factor should reproduce the roughly 20-fold advantage of bootRanges over GSC
- Both methods must be timed on identical peak sets, block/segmentation settings, and hardware for the comparison to be valid
- Results file timing_benchmark.csv should contain mean generation times for both methods side by side

## Limitations

- Shuffling or permutation schemes (as benchmarked via GSC) may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so a favorable timing result does not indicate statistical validity
- Timing comparisons are hardware-dependent; the ~20-fold speedup holds only when GSC is timed on identical peak sets and hardware as bootRanges

## Evidence

- [intro] Comparing bootRanges and GSC running time on genome-scale statistics: "Comparing bootRanges and GSC running time on genome-scale statistics"
- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [readme] bootranges_vs_GSC.R: Comparing bootRanges and GSC running time on genome-scale statistics: "[bootranges_vs_GSC.R](https://github.com/Wancen/bootRangespaper/blob/master/script/bootranges_vs_GSC.R): Comparing bootRanges and GSC running time on genome-scale statistics"
