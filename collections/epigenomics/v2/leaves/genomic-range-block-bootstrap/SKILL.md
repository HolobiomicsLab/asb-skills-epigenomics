---
name: genomic-range-block-bootstrap
description: Use when testing enrichment or correlation of genomic ranges (e.g. caQTL
  peaks vs GWAS SNPs, DEG vs differential accessibility peaks, CRE-gene correlations
  across pseudo-bulk cell types) against a null and conventional shuffling is at risk
  of an overly narrow null distribution.
when_to_use_negative:
- When the hypothesis test tolerates an unsegmented, exchangeable null — conventional
  shuffling (bootstrap.R) is what the paper compares against, and it is faster though
  it over-estimates significance (z=18.5 vs ~4)
- When inputs are not genomic ranges with a genome segmentation available — segmented
  block bootstrap requires a GRanges partition to sample blocks within matching segmentation
  states
- When comparing running time on genome-scale statistics against GSC is the goal —
  that is a separate benchmark (bootranges_vs_GSC.R), not the bootstrap itself
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Core function that performs the segmented block bootstrap of genomic ranges
    given a segmentation and block length Lb, returning a BootRanges object
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges, installed and loaded to run the
    block bootstrap
  repo: https://bioconductor.org/packages/nullranges
- name: bootstrap.R
  role: Paper script performing conventional shuffling, used as the baseline null
    whose z-score (18.5) is contrasted with segmented block bootstrap (~4)
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
- name: var.R
  role: Paper script computing the variance of the rate of overlaps and the z score
    from the bootstrapped null sets
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:f0f12e220c5316da36f17ad382d67136a6792c59641b1058b8e12411a64ef9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - bootstrap.R
  - nullranges
  - var.R
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# genomic-range-block-bootstrap

## Summary

Generates null sets of genomic ranges via segmented block bootstrap (bootRanges in the R/Bioconductor package nullranges), sampling blocks of length Lb within genome-segmentation states so that null ranges preserve local genomic correlation structure and yield more reliable null distributions than shuffling or permutation.

## When to use

Use when testing enrichment or correlation of genomic ranges (e.g. caQTL peaks vs GWAS SNPs, DEG vs differential accessibility peaks, CRE-gene correlations across pseudo-bulk cell types) against a null and conventional shuffling is at risk of an overly narrow null distribution.

## When NOT to use

- When the hypothesis test tolerates an unsegmented, exchangeable null — conventional shuffling (bootstrap.R) is what the paper compares against, and it is faster though it over-estimates significance (z=18.5 vs ~4)
- When inputs are not genomic ranges with a genome segmentation available — segmented block bootstrap requires a GRanges partition to sample blocks within matching segmentation states
- When comparing running time on genome-scale statistics against GSC is the goal — that is a separate benchmark (bootranges_vs_GSC.R), not the bootstrap itself

## Inputs

- Genome segmentation as a GRanges partition of the genome (e.g. bins stratified by gene density or pre-defined states, produced by GRanges_construct.R)
- Query/observed genomic ranges as GRanges objects (e.g. human liver caQTL peaks, total cholesterol GWAS SNPs)
- Block length Lb (e.g. 300 kb, 600 kb) and number of bootstrap iterations R

## Outputs

- BootRanges object: a GRanges containing R bootstrapped copies of the query ranges, concatenated across iterations with iteration indices
- bootranges.rds saved via saveRDS for downstream overlap statistics
- Overlap rates, their variance, and z-scores (e.g. liver_chol_zscores.csv) computed by var.R

## How to apply

Construct GRanges objects for the observed feature sets (e.g. via the paper's GRanges_construct.R preprocessing) and a genome segmentation (a GRanges partition such as bins stratified by gene density or pre-defined states). Choose a block length Lb (the liver analysis evaluated 300 kb and 600 kb) and number of bootstrap iterations R, then call bootRanges from nullranges so blocks are sampled within segments of the same segmentation state and placed to preserve local genomic correlation structure. bootRanges returns a BootRanges GRanges that concatenates ranges across the R bootstrap iterations with associated iteration indices, which can be saved with saveRDS (e.g. bootranges.rds) for downstream scripts. In the paper's pipeline, segBootranges.R performs the bootstrap, var.R computes the variance of the overlap rate and z score, and bootstrap.R provides a conventional shuffling null for comparison. Note that shuffling yielded an inflated z-score (z=18.5) versus z~4 for segmented block bootstrap in the liver caQTL/total-cholesterol-SNP analysis, so use the block-bootstrap null for significance assessment.

## Related tools

- **bootRanges** (Core function that performs the segmented block bootstrap of genomic ranges given a segmentation and block length Lb, returning a BootRanges object) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges, installed and loaded to run the block bootstrap) — https://bioconductor.org/packages/nullranges
- **bootstrap.R** (Paper script performing conventional shuffling, used as the baseline null whose z-score (18.5) is contrasted with segmented block bootstrap (~4)) — https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
- **var.R** (Paper script computing the variance of the rate of overlaps and the z score from the bootstrapped null sets) — https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md

## Examples

```
segBootranges.R (perform bootRanges) -> var.R (Variance of the rate of overlaps and z score); e.g. saveRDS(bootRanges(query, seg, blockLength=3e5, R=...), 'bootranges.rds')
```

## Evaluation signals

- Returned object is a BootRanges GRanges containing R bootstrapped copies of the query ranges partitioned by the supplied segmentation
- In the liver caQTL/total-cholesterol analysis, the segmented block-bootstrap z-score falls near ~4 (vs z=18.5 for shuffling), confirming shuffling gives an overly narrow null distribution
- In the 10x single-cell multiome application, observed mean gene-peak correlation (~0.33) substantially exceeds the bootstrap null mean (~0.007)
- Sampled blocks are drawn within segments of the same segmentation state, preserving local genomic correlation structure of the original ranges
- Output saved to bootranges.rds is consumable by var.R for variance and z-score computation

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so they are not interchangeable with the block bootstrap
- Segmented block bootstrap depends on the quality/choice of the genome segmentation and the block length Lb; the liver analysis tested multiple segmentation methods at Lb = 300 kb and 600 kb
- For genome-scale statistics, bootRanges' running time was benchmarked against GSC, so computational cost may matter for very large analyses

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
- [readme] *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score): "*segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
