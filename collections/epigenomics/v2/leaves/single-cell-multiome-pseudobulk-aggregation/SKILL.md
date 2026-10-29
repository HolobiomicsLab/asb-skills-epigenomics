---
name: single-cell-multiome-pseudobulk-aggregation
description: Use when single-cell multiome ATAC+RNA counts need to be aggregated per
  cell type into pseudo-bulk profiles before computing gene-peak correlations, as
  in bootRanges multiomicsCor.R (14 pseudo-bulk cell types).
when_to_use_negative:
- Input is already aggregated pseudo-bulk or bulk count data rather than single-cell
  multiome counts
- The analysis goal is range-overlap enrichment (e.g. caQTL overlap rates or DEG/DA
  peak overlaps) rather than gene-peak correlation, where the liver (GRanges_construct.R
  -> segBootranges.R) or macrophage.R workflows apply instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: bootRanges
  role: Generates segmented block bootstrapped null sets of genomic ranges used to
    build the null distribution of gene-peak mean correlations
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package that hosts bootRanges for null range generation
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:8086cb9a0891e100f7a67454a10e58725d7d8d7587791ef2bcff5651e9530866
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-multiome-pseudobulk-aggregation

## Summary

Aggregates 10x Genomics Single Cell Multiome ATAC + Gene Expression single-cell counts into pseudo-bulk profiles per cell type so that gene-peak log-count correlations can be computed across cell types and tested against bootRanges block-bootstrap null ranges.

## When to use

Use when single-cell multiome ATAC+RNA counts need to be aggregated per cell type into pseudo-bulk profiles before computing gene-peak correlations, as in bootRanges multiomicsCor.R (14 pseudo-bulk cell types).

## When NOT to use

- Input is already aggregated pseudo-bulk or bulk count data rather than single-cell multiome counts
- The analysis goal is range-overlap enrichment (e.g. caQTL overlap rates or DEG/DA peak overlaps) rather than gene-peak correlation, where the liver (GRanges_construct.R -> segBootranges.R) or macrophage.R workflows apply instead

## Inputs

- 10x Genomics Single Cell Multiome ATAC + Gene Expression count matrices (peak and gene modalities)
- Cell-type labels used to aggregate single cells into pseudo-bulk profiles
- Genomic ranges for peaks and genes (GRanges objects)

## Outputs

- Per-pair gene-peak correlations across the 14 pseudo-bulk cell types
- Observed mean gene-peak correlation (approximately 0.33) and bootstrap null mean correlation (approximately 0.007)
- multiomics_correlation.csv with per-pair correlations and the mean-comparison summary

## How to apply

Load the 10x Genomics Single Cell Multiome ATAC + Gene Expression counts and aggregate single cells into 14 pseudo-bulk cell types, collapsing per-cell log-counts into one pseudo-bulk profile per cell type for peaks and genes separately. Compute per-pair Spearman/Pearson correlations between ATAC peak log-counts and RNA gene log-counts across the 14 pseudo-bulk profiles, recording the observed mean correlation (expected around 0.33 in the paper). Generate null sets of genomic ranges with the bootRanges segmented block bootstrap from the R/Bioconductor package nullranges, which preserves local genomic correlation structure, and recompute peak-gene log-count correlations against each bootstrapped null set to build a null distribution of mean correlations (null mean around 0.007). Compare the observed mean to the bootstrap null mean and write per-pair correlations plus the mean-comparison summary to multiomics_correlation.csv. The rationale is that pseudo-bulk aggregation stabilizes per-cell-type signals while block bootstrap avoids the overly narrow null distributions that simple shuffling or permutation schemes produce.

## Related tools

- **bootRanges** (Generates segmented block bootstrapped null sets of genomic ranges used to build the null distribution of gene-peak mean correlations) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package that hosts bootRanges for null range generation) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript script/multiomicsCor.R
```

## Evaluation signals

- Observed mean gene-peak correlation across the 14 pseudo-bulk cell types is approximately 0.33
- Bootstrap null mean correlation is approximately 0.007
- Observed mean correlation substantially exceeds the block-bootstrap null, demonstrating that observed CRE-gene correlations exceed the null
- multiomics_correlation.csv is produced containing per-pair correlations and the mean-comparison summary
- Null ranges are generated by segmented block bootstrap (preserving local genomic correlation structure), not simple shuffling or permutation

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so the block bootstrap must be used
- Validation is judged against the paper's approximate reference values (0.33 observed vs 0.007 null); datasets differing in cell types or pseudo-bulk counts will not reproduce these exact numbers

## Evidence

- [readme] Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics: "multiomicsCor.R (https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R): Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics"
- [abstract] bootRanges in complex correlation analyses: "It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types"
- [abstract] Block bootstrap preserves local genomic correlation structure: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions"
- [abstract] Shuffling schemes produce overly narrow nulls: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] bootRanges availability in nullranges: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges"
