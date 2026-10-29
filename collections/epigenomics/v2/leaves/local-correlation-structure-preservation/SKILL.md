---
name: local-correlation-structure-preservation
description: Use when testing enrichment or overlap of genomic ranges (e.g. caQTLs/SNPs,
  CREs, ATAC peaks) against a segmentation where naive shuffling would overstate significance
  due to spatial correlation.
when_to_use_negative:
- When ranges are independent of genomic location such that simple shuffling or permutation
  suffices — though the paper shows shuffling may give overly narrow null distributions
  and over-estimated significance, bootstrapping adds cost unnecessary if no spatial
  correlation exists
- When no meaningful genome segmentation is available and block placement cannot be
  constrained to like-state segments
- When the goal is conventional (non-segmented) permutation-based overlap testing
  — bootstrap.R in the repo covers that baseline instead
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Core function performing the segmented block bootstrap that produces the BootRanges
    object preserving local genomic correlation structure
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges and its supporting infrastructure
    for null range generation
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:88061d8fdbc1718b84894f01d993e4826899055e27a29d2e6a184b13de3868d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# local-correlation-structure-preservation

## Summary

Generates null sets of genomic ranges via segmented block bootstrap (bootRanges in the nullranges R/Bioconductor package) so that resampled ranges preserve the local genomic correlation structure of the original query, producing more reliable null distributions for enrichment testing than shuffling or permutation.

## When to use

Use when testing enrichment or overlap of genomic ranges (e.g. caQTLs/SNPs, CREs, ATAC peaks) against a segmentation where naive shuffling would overstate significance due to spatial correlation.

## When NOT to use

- When ranges are independent of genomic location such that simple shuffling or permutation suffices — though the paper shows shuffling may give overly narrow null distributions and over-estimated significance, bootstrapping adds cost unnecessary if no spatial correlation exists
- When no meaningful genome segmentation is available and block placement cannot be constrained to like-state segments
- When the goal is conventional (non-segmented) permutation-based overlap testing — bootstrap.R in the repo covers that baseline instead

## Inputs

- Query GRanges to be bootstrapped (e.g. caQTL SNP ranges, CREs, differential accessibility peaks)
- Genome segmentation GRanges partitioning the genome (e.g. by gene density or state, from GRanges_construct.R preprocessing)
- Block length Lb and number of bootstrap iterations R

## Outputs

- BootRanges object: a GRanges containing R bootstrapped copies of the query ranges concatenated with iteration indices
- bootranges.rds (saved with saveRDS) consumed by var.R for variance of the rate of overlaps and z score

## How to apply

Construct a genome segmentation as a GRanges partition (e.g. bins stratified by gene density or pre-defined states) with GRanges_construct.R, then call bootRanges from nullranges with the query ranges, a block length Lb, and R bootstrap iterations; blocks are sampled within segments of the same state and placed so local genomic correlation structure is preserved. The function returns a BootRanges object — a GRanges concatenating ranges across the R bootstrap iterations with associated iteration indices — which is saved with saveRDS (e.g. bootranges.rds) for downstream scripts like var.R, which computes the variance of the rate of overlaps and the z score. Larger Lb retains more local correlation; blocks are only drawn within segments sharing a segmentation state, so segmentation quality directly controls null fidelity. As a negative control or baseline, run conventional shuffling via bootstrap.R to see the narrower null distributions the paper warns about.

## Related tools

- **bootRanges** (Core function performing the segmented block bootstrap that produces the BootRanges object preserving local genomic correlation structure) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges and its supporting infrastructure for null range generation) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript segBootranges.R  # performs bootRanges on the segmentation/query GRanges, saving bootranges.rds for var.R
```

## Evaluation signals

- Returned object is a BootRanges GRanges containing R bootstrapped copies of the query ranges partitioned by the supplied segmentation
- Bootstrapped ranges fall within segments of the same segmentation state as the original placements (blocks are sampled across like-state segments)
- Null distribution of the overlap statistic from var.R is wider/more reliable than the conventional shuffling baseline from bootstrap.R (shuffling yields overly narrow nulls)
- bootranges.rds loads via readRDS and includes iteration indices for downstream overlap statistics
- Pipeline order matches GRanges_construct.R -> segBootranges.R -> var.R as documented in the repo

## Limitations

- Shuffling or permutation schemes (e.g. the bootstrap.R baseline) may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so results from those baselines should not be treated as equivalent
- Null quality depends on the segmentation: blocks are only sampled within same-state segments, so a poor segmentation undermines correlation preservation
- Conventional shuffling comparisons on genome-scale statistics were benchmarked against GSC (bootranges_vs_GSC.R); runtime considerations apply at genome scale

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
- [readme] GRanges_construct.R(Preprocess) -> segBootranges.R(perform bootRanges) -> var.R(Variance of the rate of overlaps and z score): "*GRanges_construct.R*(Preprocess) -> *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
