---
name: null-range-generation-for-enrichment-testing
description: Use when testing enrichment/overlap of genomic ranges (e.g. caQTLs, CREs,
  differential peaks) where naive shuffling or permutation would over-estimate statistical
  significance due to genomic correlation.
when_to_use_negative:
- When the analysis goal permits conventional shuffling/permutation and genomic correlation
  structure is not a concern — the paper shows shuffling can yield overly narrow null
  distributions, but bootRanges is specifically the alternative for correlated genomic
  data
- When benchmarking running time against GSC — this skill covers null range generation,
  not the bootranges_vs_GSC.R timing comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0085
tools:
- name: bootRanges
  role: Performs segmented block bootstrap of query ranges within same-state segments
    given block length Lb, producing BootRanges null sets
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting the bootRanges functions for generating block
    bootstrapped genomic ranges
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Alternative genome-scale tool compared against bootRanges for running time
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:3296b81a8528eae7a3d67e66c6b7c1b6b3257f159a5d8946434ded3f42c379f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# null-range-generation-for-enrichment-testing

## Summary

Generates null sets of genomic ranges via segmented block bootstrap (bootRanges from the nullranges R/Bioconductor package) to test enrichment of query ranges against a genome segmentation, preserving local genomic correlation structure instead of using shuffling or permutation.

## When to use

Use when testing enrichment/overlap of genomic ranges (e.g. caQTLs, CREs, differential peaks) where naive shuffling or permutation would over-estimate statistical significance due to genomic correlation.

## When NOT to use

- When the analysis goal permits conventional shuffling/permutation and genomic correlation structure is not a concern — the paper shows shuffling can yield overly narrow null distributions, but bootRanges is specifically the alternative for correlated genomic data
- When benchmarking running time against GSC — this skill covers null range generation, not the bootranges_vs_GSC.R timing comparison

## Inputs

- Genome segmentation as a GRanges partition of the genome (bins stratified by gene density or pre-defined states)
- Query genomic ranges to be bootstrapped (e.g. caQTL SNP ranges, differential accessibility peaks, DEGs)
- Block length Lb
- Number of bootstrap iterations R

## Outputs

- BootRanges object (GRanges concatenating R bootstrapped copies of the query ranges with iteration indices)
- bootranges.rds file for downstream variance/z-score computation
- Overlap rates/counts and z scores against the null distribution

## How to apply

Construct the genome segmentation as a GRanges partition of the genome (e.g. bins stratified by gene density or pre-defined states) with GRanges_construct.R in preprocessing. Call bootRanges from nullranges, passing the query ranges, the segmentation, block length Lb, and number of bootstrap iterations R so that blocks are sampled across segments in the same segmentation state, preserving local genomic correlation structure. Concatenate the bootstrapped ranges across R iterations into a single BootRanges object with iteration indices, and save it with saveRDS (e.g. bootranges.rds) for downstream overlap statistics. Compute variance of the rate of overlaps and z score in var.R; optionally compare against conventional shuffling (bootstrap.R) to show the permutation null is overly narrow. For complex analyses, extend to CRE-gene correlations across cell types (multiomicsCor.R) or DEG logFC threshold optimization via generalized splines (macrophage.R).

## Related tools

- **bootRanges** (Performs segmented block bootstrap of query ranges within same-state segments given block length Lb, producing BootRanges null sets) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting the bootRanges functions for generating block bootstrapped genomic ranges) — https://bioconductor.org/packages/nullranges
- **GSC** (Alternative genome-scale tool compared against bootRanges for running time)

## Examples

```
library(nullranges); bootranges <- bootRanges(query, seg = segmentation, L_b = Lb, R = R_iters); saveRDS(bootranges, 'bootranges.rds')
```

## Evaluation signals

- The returned object is a BootRanges GRanges containing R bootstrapped copies of the query ranges partitioned by the supplied segmentation
- Bootstrapped range sets preserve local genomic correlation structure of the original ranges
- Null distribution from block bootstrap is wider (more reliable) than the overly narrow null from shuffling/permutation
- bootranges.rds exists and is consumable by var.R for variance of the rate of overlaps and z score
- Sampled blocks are drawn within segments of the same segmentation state

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance (motivating the block bootstrap, but also a failure mode if the practitioner defaults to permutation)
- Requires a genome segmentation; without a valid GRanges partition of the genome, segmented block bootstrap cannot be applied

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [readme] *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score): "*segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
