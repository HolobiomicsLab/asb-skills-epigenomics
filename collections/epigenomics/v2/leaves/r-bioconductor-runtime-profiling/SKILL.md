---
name: r-bioconductor-runtime-profiling
description: Use when you must compare runtime of two genomic-range null-generation
  methods (bootRanges vs GSC) on identical GRanges inputs, e.g. ENCODE ChIP-seq peaks,
  to quantify a speedup factor on genome-scale statistics.
when_to_use_negative:
- The goal is the statistical validity of null distributions rather than speed — the
  paper shows permutation/shuffling schemes can produce overly narrow test statistic
  null distributions and over-estimate significance, which timing alone does not address.
- The input is not GRanges-style genomic ranges (e.g. an expression matrix or feature
  table) — bootRanges and GSC operate on genomic range sets.
- Only one method is of interest and no cross-method comparison or speedup factor
  is needed.
tools:
- name: bootRanges
  role: Block-bootstraps genomic ranges to generate null range sets; its generation
    and overlap-computation runtimes are measured in the benchmark
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package that hosts bootRanges; installed from Bioconductor
    for the timing analysis
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Permutation-based genomic shuffling baseline whose mean runtime is compared
    against bootRanges
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:790ef640a308774efbbaf49e46c466805247f5c4a525a82ab818cb9412e5e983
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# r-bioconductor-runtime-profiling

## Summary

Benchmark the wall-clock runtime of the R/Bioconductor bootRanges block bootstrapper against the GSC permutation-based shuffler on the same GRanges peak sets and hardware, quantifying the speedup of genome-scale null-range generation.

## When to use

Use when you must compare runtime of two genomic-range null-generation methods (bootRanges vs GSC) on identical GRanges inputs, e.g. ENCODE ChIP-seq peaks, to quantify a speedup factor on genome-scale statistics.

## When NOT to use

- The goal is the statistical validity of null distributions rather than speed — the paper shows permutation/shuffling schemes can produce overly narrow test statistic null distributions and over-estimate significance, which timing alone does not address.
- The input is not GRanges-style genomic ranges (e.g. an expression matrix or feature table) — bootRanges and GSC operate on genomic range sets.
- Only one method is of interest and no cross-method comparison or speedup factor is needed.

## Inputs

- GRanges objects of genomic peaks or ranges (e.g. ENCODE kidney and bladder ChIP-seq peak sets)
- Segmentation-derived block lengths for segmental block bootstrapping
- Target features for the overlap-computation timing variant

## Outputs

- Mean runtime of bootRanges block-bootstrap generation (seconds)
- Mean runtime of bootRanges generation plus overlap computation (seconds)
- Mean runtime of GSC permutation-based shuffling (seconds)
- Speedup factor and side-by-side timing table written to timing_benchmark.csv

## How to apply

Load the peak sets (e.g. ENCODE kidney and bladder ChIP-seq peaks) as GRanges objects, preprocessing them with the GRanges_construct.R step from the bootRangespaper repository. Time bootRanges block-bootstrap generation of null range sets using segmental blocks matched to segmentation-derived block lengths, recording the mean over repeated bootstrap iterations, then re-time with the downstream overlap computation included to capture both the generation-only and generation-plus-overlap figures. Time GSC permutation-based shuffling on the identical peak sets, hardware, and scale. Tabulate mean generation times side by side in a CSV (timing_benchmark.csv) and compute the speedup factor; the paper's benchmark relationship is bootRanges ~0.30 s (0.37 s with overlaps) versus GSC 7.56 s, roughly a 20-fold speedup.

## Related tools

- **bootRanges** (Block-bootstraps genomic ranges to generate null range sets; its generation and overlap-computation runtimes are measured in the benchmark) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package that hosts bootRanges; installed from Bioconductor for the timing analysis) — https://bioconductor.org/packages/nullranges
- **GSC** (Permutation-based genomic shuffling baseline whose mean runtime is compared against bootRanges)

## Examples

```
Rscript script/bootranges_vs_GSC.R
```

## Evaluation signals

- The recorded bootRanges mean generation time should be ~0.30 s per block bootstrap of the genome, and ~0.37 s when overlap computation is included.
- The recorded GSC mean runtime on the identical peak sets and hardware should be ~7.56 s, i.e. roughly a 20-fold speedup for bootRanges.
- Both methods must be timed on the same GRanges peak sets, the same hardware, and the same genome scale for the speedup factor to be valid.
- Mean timings over repeated bootstrap iterations should be stable across repeats, not single-shot measurements.
- The results table (timing_benchmark.csv) should contain both methods' mean times and the computed speedup factor side by side.

## Limitations

- Timings are hardware-dependent; the reported 0.30 s / 0.37 s vs 7.56 s relationship only reproduces on comparable hardware and the same genome-scale peak sets.
- The benchmark measures runtime only and does not evaluate the statistical adequacy of the null distributions; the paper notes permutation schemes may over-estimate significance.
- The comparison is specific to genome-scale statistics; runtimes on small peak sets may not show the same speedup.

## Evidence

- [intro] Comparing bootRanges and GSC running time on genome-scale statistics: "Compare bootRanges and GSC running time on genome-scale statistics"
- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [readme] bootranges_vs_GSC.R comparison script: "bootranges_vs_GSC.R(https://github.com/Wancen/bootRangespaper/blob/master/script/bootranges_vs_GSC.R): Comparing bootRanges and GSC running time on genome-scale statistics"
