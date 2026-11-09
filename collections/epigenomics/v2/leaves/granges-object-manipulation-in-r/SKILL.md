---
name: granges-object-manipulation-in-r
description: Use when genomic features (SNP/caQTL ranges, peaks, a genome segmentation)
  must be represented as GRanges and block-bootstrapped with bootRanges to build null
  ranges for overlap enrichment analysis.
when_to_use_negative:
- Shuffling or permutation is specifically desired as the null model — bootRanges'
  block bootstrap exists because shuffling/permutation can give overly narrow null
  distributions and over-estimated significance
- The analysis goal is running-time comparison against GSC rather than generating
  null ranges (use bootranges_vs_GSC.R context instead)
- Input data is not genomic ranges and has not been preprocessed into GRanges form
  (preprocessing via GRanges_construct.R must come first)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: bootRanges
  role: Generates block bootstrapped genomic ranges representing the null hypothesis
    in enrichment analysis; produces the BootRanges object
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges, called on the segmentation and
    query GRanges with Lb and R parameters
  repo: https://bioconductor.org/packages/nullranges
- name: GenomicRanges
  role: R/Bioconductor infrastructure used to load and construct the genome segmentation
    and query ranges as GRanges objects
- name: GSC
  role: Alternative genome-scale null-range tool used as a running-time comparison
    baseline for bootRanges
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:95ae5bcd623db7f39a3de8392fdcca2974740d59fb251c7ee3a15c82001e4bbc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - GenomicRanges
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# granges-object-manipulation-in-r

## Summary

Construct and manipulate GRanges objects in R with GenomicRanges to represent genomic features (segmentations, SNPs, peaks) as inputs to nullranges/bootRanges, producing a BootRanges object via segmented block bootstrap.

## When to use

Use when genomic features (SNP/caQTL ranges, peaks, a genome segmentation) must be represented as GRanges and block-bootstrapped with bootRanges to build null ranges for overlap enrichment analysis.

## When NOT to use

- Shuffling or permutation is specifically desired as the null model — bootRanges' block bootstrap exists because shuffling/permutation can give overly narrow null distributions and over-estimated significance
- The analysis goal is running-time comparison against GSC rather than generating null ranges (use bootranges_vs_GSC.R context instead)
- Input data is not genomic ranges and has not been preprocessed into GRanges form (preprocessing via GRanges_construct.R must come first)

## Inputs

- Genome segmentation as a GRanges partition of the genome (e.g. bins stratified by gene density or pre-defined states), produced by GRanges_construct.R
- Query genomic ranges to be bootstrapped (e.g. caQTL SNP ranges from human liver tissue, ATAC-seq peaks) as GRanges
- Block bootstrap parameters: block length Lb and number of bootstrap iterations R

## Outputs

- A BootRanges GRanges object concatenating R bootstrapped copies of the query ranges partitioned by the segmentation, with associated iteration indices
- bootranges.rds (saved via saveRDS) consumed by var.R for variance of overlap rate and z score

## How to apply

First preprocess genomic features into GRanges objects with GenomicRanges in R (e.g. GRanges_construct.R builds a genome segmentation such as bins stratified by gene density, plus query ranges like liver caQTL SNPs). Then call bootRanges from the nullranges R/Bioconductor package, passing the segmentation and a block length Lb and number of iterations R, so that blocks are sampled within segments of the same segmentation state, preserving local genomic correlation structure. bootRanges concatenates ranges across the R bootstrap iterations into a single BootRanges object carrying iteration indices. Save the result with saveRDS to bootranges.rds for downstream scripts such as var.R, which computes the variance of the rate of overlaps and the z score. As a baseline contrast, the same workflow can run conventional shuffling (bootstrap.R), since shuffling or permutation may produce overly narrow null distributions and over-estimated significance.

## Related tools

- **bootRanges** (Generates block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis; produces the BootRanges object) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges, called on the segmentation and query GRanges with Lb and R parameters) — https://bioconductor.org/packages/nullranges
- **GenomicRanges** (R/Bioconductor infrastructure used to load and construct the genome segmentation and query ranges as GRanges objects)
- **GSC** (Alternative genome-scale null-range tool used as a running-time comparison baseline for bootRanges)

## Examples

```
library(nullranges); br <- bootRanges(query, seg, L=5e6, R=100); saveRDS(br, "bootranges.rds")
```

## Evaluation signals

- The returned object is a BootRanges GRanges containing R bootstrapped copies of the query ranges partitioned by the supplied segmentation
- Sampled blocks are drawn within segments of the same segmentation state, preserving local genomic correlation structure (contrast with conventional shuffling via bootstrap.R)
- The BootRanges object carries iteration indices so downstream overlap statistics can be computed per bootstrap iteration
- bootranges.rds is present and consumable by var.R, which computes the variance of the rate of overlaps and the z score
- Null distribution width is not overly narrow relative to a shuffling/permutation baseline, avoiding over-estimated statistical significance

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance — one reason the block bootstrap is preferred
- Requires a genome segmentation and specification of block length Lb; block sampling is constrained to segments in the same segmentation state
- Running time can be compared against GSC on genome-scale statistics, implying performance is a consideration for very large sets

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] Block bootstrap creates new range sets that preserve local genomic correlation structure and generate more reliable null distributions: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [readme] GRanges_construct.R(Preprocess) -> segBootranges.R(perform bootRanges) -> var.R(Variance of the rate of overlaps and z score): "*GRanges_construct.R*(Preprocess) -> *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
