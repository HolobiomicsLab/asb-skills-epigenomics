---
name: null-distribution-hypothesis-testing
description: Use when testing whether genomic ranges (peaks, QTLs, CREs) show significant
  enrichment or correlation with features (genes, cell types) and a null that preserves
  local genomic correlation structure is required.
when_to_use_negative:
- When simple random shuffling/permutation of ranges is sufficient and computational
  cost must be minimized — though note shuffling may yield overly narrow null distributions
  and over-estimated significance
- When the input is not genomic ranges or range-associated statistics (e.g., a feature
  table without coordinates), since bootRanges requires GRanges input
edam_operation: http://edamontology.org/operation_3224
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: bootRanges
  role: Generates block-bootstrapped null sets of genomic ranges preserving local
    genomic correlation structure for enrichment and correlation hypothesis testing
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges functions used to generate null
    range sets
  repo: https://bioconductor.org/packages/nullranges
- name: multiomicsCor.R
  role: Repository script that derives the mean correlation from the Single Cell Multiome
    ATAC + Gene Expression assay by 10x Genomics
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:55fd48f95c8d539637de5091e6bc69b51da27a5b5cb1de2bffd1788e04d7b939
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - multiomicsCor.R
  - nullranges
  edam_operation: http://edamontology.org/operation_3224
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# null-distribution-hypothesis-testing

## Summary

Generates null sets of genomic ranges via segmented block bootstrap (bootRanges/nullranges) to test whether observed range-based statistics — overlap enrichment or cross-omic correlations — exceed a genomic-structure-aware null. Used here to show observed gene-peak correlations in 10x single-cell multiome data (0.33) far exceed the bootstrap null (0.007).

## When to use

Use when testing whether genomic ranges (peaks, QTLs, CREs) show significant enrichment or correlation with features (genes, cell types) and a null that preserves local genomic correlation structure is required.

## When NOT to use

- When simple random shuffling/permutation of ranges is sufficient and computational cost must be minimized — though note shuffling may yield overly narrow null distributions and over-estimated significance
- When the input is not genomic ranges or range-associated statistics (e.g., a feature table without coordinates), since bootRanges requires GRanges input

## Inputs

- 10x Genomics Single Cell Multiome ATAC + Gene Expression count matrices (ATAC peaks and RNA genes)
- Genomic ranges (GRanges objects) for peaks, QTLs, or cis-regulatory elements
- Pseudo-bulk cell-type aggregations (here 14 pseudo-bulk cell types)
- Genomic segmentation for block bootstrap

## Outputs

- Null sets of block-bootstrapped genomic ranges
- Per-pair observed and null correlations between peak log-counts and gene log-counts
- Mean-comparison summary (observed mean correlation 0.33 vs bootstrap null mean 0.007) written to multiomics_correlation.csv
- Variance of the rate of overlaps and z score

## How to apply

Construct GRanges objects for the observed ranges and features (e.g., preprocess with GRanges_construct.R), then generate null range sets with bootRanges' segmented block bootstrap, which preserves local genomic correlation structure rather than shuffling randomly — the article shows shuffling/permutation can give overly narrow nulls and inflated significance. For correlation analyses, aggregate single cells into pseudo-bulk profiles (here 14 pseudo-bulk cell types from 10x Single Cell Multiome ATAC + Gene Expression data), compute per-pair Spearman/Pearson correlations between ATAC peak log-counts and RNA gene log-counts across pseudo-bulk profiles, and recompute these correlations against each bootstrapped null range set to build a null distribution of mean correlations. Compare the observed statistic to the null (e.g., observed mean correlation 0.33 vs bootstrap null mean 0.007) and record per-pair values and the mean comparison. The same framework supports overlap-rate testing with variance/z-score computation (var.R) and threshold optimization via generalized splines on DEG logFC.

## Related tools

- **bootRanges** (Generates block-bootstrapped null sets of genomic ranges preserving local genomic correlation structure for enrichment and correlation hypothesis testing) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges functions used to generate null range sets) — https://bioconductor.org/packages/nullranges
- **multiomicsCor.R** (Repository script that derives the mean correlation from the Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics) — https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R

## Examples

```
Rscript script/multiomicsCor.R
```

## Evaluation signals

- Observed mean gene-peak correlation across the 14 pseudo-bulk cell types is approximately 0.33
- Bootstrap null mean correlation is approximately 0.007, demonstrating observed CRE-gene correlations substantially exceed the block-bootstrap null
- Null distributions are wider/more reliable than those from conventional shuffling, avoiding over-estimated significance
- Per-pair correlations and the mean-comparison summary are written to multiomics_correlation.csv with a valid schema
- Overlap analyses report variance of the rate of overlaps and a z score consistent with block-bootstrapped nulls

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance — a key reason the block bootstrap is preferred
- Conventional shuffling (bootstrap.R) is retained only as a comparison baseline; results from it should not be treated as the primary null

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types: "It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types or providing optimized thresholds, e.g. log fold change"
- [readme] multiomicsCor.R (derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay): "Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics"
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
