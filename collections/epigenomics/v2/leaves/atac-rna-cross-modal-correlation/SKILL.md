---
name: atac-rna-cross-modal-correlation
description: Use when single-cell multiome ATAC+RNA data has been aggregated into
  pseudo-bulk cell types and the goal is to test whether observed peak–gene correlations
  exceed a genomic-structure-aware null.
when_to_use_negative:
- The analysis is a simple overlap/enrichment test between two range sets rather than
  a cross-modal correlation — use the caQTL-overlap or DEG/peak-overlap variant of
  bootRanges instead
- The question requires shuffling-based nulls for comparison purposes only; note the
  paper warns shuffling/permutation yields overly narrow nulls and over-estimated
  significance
- Input data lacks paired ATAC and RNA measurements from the same cells, so peak–gene
  correlation across modalities is not computable
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0769
tools:
- name: bootRanges
  role: Generates segmented block-bootstrapped null sets of genomic ranges that preserve
    local genomic correlation structure, used to build the null distribution of mean
    peak–gene correlations
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package in which bootRanges is hosted and invoked for null
    range generation
  repo: https://bioconductor.org/packages/nullranges
- name: multiomicsCor.R
  role: Analysis script from the bootRangespaper repository that derives the mean
    correlation from the Single Cell Multiome ATAC + Gene Expression assay
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:b055ea0ac9b27c1887995590beadfe5ac7e3ead831ac1be3de06150d8e450b49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - multiomicsCor.R
  - nullranges
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# atac-rna-cross-modal-correlation

## Summary

Quantifies cis-regulatory element (CRE)–gene relationships by correlating ATAC peak log-counts with RNA gene log-counts across pseudo-bulk cell types from a 10x Genomics Single Cell Multiome ATAC + Gene Expression assay, and assesses significance against block-bootstrap null ranges generated with bootRanges (nullranges R/Bioconductor package).

## When to use

Use when single-cell multiome ATAC+RNA data has been aggregated into pseudo-bulk cell types and the goal is to test whether observed peak–gene correlations exceed a genomic-structure-aware null.

## When NOT to use

- The analysis is a simple overlap/enrichment test between two range sets rather than a cross-modal correlation — use the caQTL-overlap or DEG/peak-overlap variant of bootRanges instead
- The question requires shuffling-based nulls for comparison purposes only; note the paper warns shuffling/permutation yields overly narrow nulls and over-estimated significance
- Input data lacks paired ATAC and RNA measurements from the same cells, so peak–gene correlation across modalities is not computable

## Inputs

- 10x Genomics Single Cell Multiome ATAC + Gene Expression count matrices (ATAC peak counts and RNA gene counts)
- Cell-type annotations for aggregating single cells into pseudo-bulk profiles (14 pseudo-bulk cell types in the paper)
- Genomic ranges (GRanges) of ATAC peaks and gene coordinates

## Outputs

- Per-pair peak–gene correlation values across pseudo-bulk cell types
- Observed mean gene–peak correlation (approximately 0.33 in the paper's application)
- Bootstrap null distribution of mean correlations (mean approximately 0.007)
- Mean-comparison summary written to multiomics_correlation.csv

## How to apply

Aggregate single cells from the 10x Single Cell Multiome ATAC + Gene Expression assay into pseudo-bulk profiles per cell type (14 in the paper's application), then compute per-pair Spearman/Pearson correlations between ATAC peak log-counts and RNA gene log-counts across the pseudo-bulk profiles to obtain an observed mean correlation. Do NOT assess significance by simple shuffling or permutation of peak ranges, which the paper shows produces overly narrow null distributions and over-estimates statistical significance; instead generate null peak sets with bootRanges' segmented block bootstrap, which preserves local genomic correlation structure. Recompute peak–gene log-count correlations against each bootstrapped null range set to build the null distribution of mean correlations. In the paper's application (multiomicsCor.R), the observed mean correlation across all genes was 0.33 versus a bootstrap null mean of 0.007, demonstrating that CRE–gene correlations substantially exceed the null. Write per-pair correlations and the mean-comparison summary (e.g. multiomics_correlation.csv) as output.

## Related tools

- **bootRanges** (Generates segmented block-bootstrapped null sets of genomic ranges that preserve local genomic correlation structure, used to build the null distribution of mean peak–gene correlations) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package in which bootRanges is hosted and invoked for null range generation) — https://bioconductor.org/packages/nullranges
- **multiomicsCor.R** (Analysis script from the bootRangespaper repository that derives the mean correlation from the Single Cell Multiome ATAC + Gene Expression assay) — https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R

## Examples

```
Rscript script/multiomicsCor.R  # derive mean ATAC-RNA correlation from 10x Single Cell Multiome and compare to bootRanges block-bootstrap null
```

## Evaluation signals

- Observed mean gene–peak correlation across all genes is approximately 0.33
- Bootstrap null mean correlation is approximately 0.007, substantially below the observed mean
- Per-pair correlations and mean-comparison summary are present in multiomics_correlation.csv
- Null ranges are generated via segmented block bootstrap (preserving local genomic correlation structure), not naive shuffling/permutation
- Correlations are computed across 14 pseudo-bulk cell types derived from the 10x multiome assay

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so they should not replace block bootstrap for this test
- Reliability of the null depends on the block bootstrap preserving local genomic correlation structure; deviations in genome segmentation or block length choices may affect null calibration
- Results depend on correct pseudo-bulk aggregation; the paper's benchmark used 14 pseudo-bulk cell types from the 10x multiome assay

## Evidence

- [abstract] It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types: "It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types"
- [abstract] Block bootstrap preserves local genomic correlation structure: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [intro] Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics: "Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics"
- [readme] multiomicsCor.R script description from README: "[multiomicsCor.R](https://github.com/Wancen/bootRangespaper/blob/master/script/multiomicsCor.R): Derive mean correlation from Single Cell Multiome ATAC + Gene Expression assay by 10x Genomics"
