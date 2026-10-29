---
name: segmentation-stratified-range-resampling
description: Use when computing enrichment/overlap statistics between genomic query
  ranges (e.g. caQTLs, DHS peaks) and feature ranges and a null distribution that
  preserves local genomic correlation structure is needed.
when_to_use_negative:
- 'When the analysis goal is a conventional shuffled null — the paper treats ''bootstrap.R:
  perform conventional shuffling'' as a separate baseline workflow rather than segmented
  block bootstrap.'
- When no genome segmentation is available and the query ranges' local genomic correlation
  structure need not be preserved, e.g. simple uniform resampling suffices.
- When the goal is runtime benchmarking against genome-scale statistic tools — use
  bootranges_vs_GSC.R comparisons with GSC instead of treating this as a general resampling
  utility.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: 'Core function performing the segmented block bootstrap: samples blocks of
    length Lb within same-state segments and returns a BootRanges object of concatenated
    bootstrapped ranges'
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting the bootRanges functions for generation of
    block bootstrapped genomic ranges
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Baseline tool used for comparison of running time on genome-scale statistics
    against bootRanges (bootranges_vs_GSC.R)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:20e2f4abcb80a6746df3b1349dd8cb5868edb8d663c9365a7be328413beb021e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# segmentation-stratified-range-resampling

## Summary

Generates null sets of genomic ranges via segmented block bootstrap with bootRanges (R/Bioconductor package nullranges): blocks of length Lb are sampled within same-state segments of a genome segmentation, producing a BootRanges object that preserves local genomic correlation structure for reliable enrichment testing.

## When to use

Use when computing enrichment/overlap statistics between genomic query ranges (e.g. caQTLs, DHS peaks) and feature ranges and a null distribution that preserves local genomic correlation structure is needed.

## When NOT to use

- When the analysis goal is a conventional shuffled null — the paper treats 'bootstrap.R: perform conventional shuffling' as a separate baseline workflow rather than segmented block bootstrap.
- When no genome segmentation is available and the query ranges' local genomic correlation structure need not be preserved, e.g. simple uniform resampling suffices.
- When the goal is runtime benchmarking against genome-scale statistic tools — use bootranges_vs_GSC.R comparisons with GSC instead of treating this as a general resampling utility.

## Inputs

- Genome segmentation: GRanges partition of the genome (bins stratified by gene density or pre-defined states), from GRanges_construct.R preprocessing
- Query genomic ranges to be bootstrapped (GRanges, e.g. caQTL SNPs or peak ranges)
- Block bootstrap parameters: block length Lb and number of bootstrap iterations R

## Outputs

- BootRanges object: a GRanges containing R bootstrapped copies of the query ranges, concatenated across iterations with iteration indices
- bootranges.rds file written with saveRDS for consumption by var.R

## How to apply

Load a genome segmentation — a GRanges partition of the genome stratified by, e.g., gene density or pre-defined states produced by GRanges_construct.R — together with the query ranges using GenomicRanges in R. Specify the block length Lb and number of bootstrap iterations R, then call bootRanges from nullranges so that sampled blocks are drawn only within segments of the same segmentation state and placed to preserve local genomic correlation structure. bootRanges concatenates the bootstrapped ranges across the R iterations into a single BootRanges object carrying iteration indices, which is saved as bootranges.rds with saveRDS for downstream computation of the variance of the rate of overlaps and z score (var.R). Rationale: shuffling or permutation schemes may yield overly narrow test statistic null distributions and over-estimated significance, whereas the block bootstrap generates more reliable null distributions.

## Related tools

- **bootRanges** (Core function performing the segmented block bootstrap: samples blocks of length Lb within same-state segments and returns a BootRanges object of concatenated bootstrapped ranges) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting the bootRanges functions for generation of block bootstrapped genomic ranges) — https://bioconductor.org/packages/nullranges
- **GSC** (Baseline tool used for comparison of running time on genome-scale statistics against bootRanges (bootranges_vs_GSC.R))

## Examples

```
Rscript script/liver/segBootranges.R  # calls bootRanges(seg, Lb, R) from nullranges and saveRDS() to bootranges.rds
```

## Evaluation signals

- The returned object is a BootRanges GRanges (not a plain GRanges list) containing R bootstrapped copies of the query ranges.
- Sampled blocks are drawn only within segments sharing the same segmentation state as the source segment of each block.
- Bootstrapped range sets preserve local genomic correlation structure of the original ranges.
- The BootRanges object is successfully serialized to bootranges.rds via saveRDS and consumed by var.R to compute variance of the rate of overlaps and z score.
- Resulting null distribution of overlap statistics is not overly narrow (no over-estimation of statistical significance as seen with shuffling/permutation schemes).

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance — which is why segmented block bootstrap, not permutation, is used here.
- Requires a pre-computed genome segmentation (e.g. from GRanges_construct.R); without a segmentation the same-state block sampling cannot be performed.
- Performance on genome-scale statistics depends on implementation — bootRanges running time was explicitly benchmarked against GSC.

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
- [readme] segBootranges.R (perform bootRanges) -> var.R (Variance of the rate of overlaps and z score): "*GRanges_construct.R*(Preprocess) -> *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
