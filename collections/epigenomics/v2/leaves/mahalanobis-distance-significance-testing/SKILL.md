---
name: mahalanobis-distance-significance-testing
description: Use when comparing compartment scores of 100 kb (or finer, down to 10
  kb) genomic bins across two or more replicate-grouped Hi-C contact maps (e.g., mouse
  ESC vs NPC) and needing statistically significant differential compartment calls.
when_to_use_negative:
- Only a single Hi-C contact map is available — the multivariate distance requires
  two or more maps/replicates to define the score distribution
- Compartment scores have not been quantile-normalized across maps — raw PCA PC1 values
  are not comparable and Mahalanobis distances would be misleading
- Single-cell Hi-C maps are used without pseudo-bulk aggregation (>=~100 cells per
  condition) — sparsity of single-cell contact maps undermines compartment identification
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: dcHiC
  role: Implements the Mahalanobis-distance outlier test on quantile-normalized compartment
    scores, chi-square significance assignment, and IHW/FDR correction
  repo: https://github.com/ay-lab/dcHiC
- name: Fit-Hi-C
  role: Supplies per-replicate significant chromatin interaction calls (FDR < 0.05)
    whose anchors are filtered against differential compartments before per-interaction
    Mahalanobis testing
  repo: https://github.com/ay-lab/fithic
- name: HOMER
  role: Independent differential compartment scoring used to assess overlap/concordance
    with dcHiC's Mahalanobis-based calls (1355 of 3042 bins overlap, ±1 bin slack)
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:53d54c6f367f819184cbbfdfaa6a5a5216a2f941615ea0f51513e806b1fd8b13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mahalanobis-distance-significance-testing

## Summary

dcHiC's statistical core: after quantile-normalizing PC1 compartment scores across all input Hi-C maps, per-bin Mahalanobis distances (a multivariate z-score) identify bins whose compartment scores are outliers across conditions, with chi-square p-values corrected via IHW or FDR to call significant differential compartments at e.g. FDR < 0.1 (RefineY_Rconf90_FDR10).

## When to use

Use when comparing compartment scores of 100 kb (or finer, down to 10 kb) genomic bins across two or more replicate-grouped Hi-C contact maps (e.g., mouse ESC vs NPC) and needing statistically significant differential compartment calls.

## When NOT to use

- Only a single Hi-C contact map is available — the multivariate distance requires two or more maps/replicates to define the score distribution
- Compartment scores have not been quantile-normalized across maps — raw PCA PC1 values are not comparable and Mahalanobis distances would be misleading
- Single-cell Hi-C maps are used without pseudo-bulk aggregation (>=~100 cells per condition) — sparsity of single-cell contact maps undermines compartment identification

## Inputs

- Quantile-normalized PC1 compartment scores per 100 kb bin for each replicate of each condition (e.g., ESC and NPC Hi-C maps binned at 100 kb)
- Optionally, per-replicate significant interaction calls (Fit-Hi-C, FDR < 0.05) or chromatin loops (HiCCUPS/Mustache) filtered to anchors overlapping significant differential compartments

## Outputs

- Per-bin significance table with Mahalanobis distances, chi-square p-values, IHW/FDR-adjusted p-values, and A->B / B->A / within-compartment flip annotations (results_table.tsv)
- Differential interactions table with adjusted p-values for loops anchored in significant differential compartments (differential_interactions.tsv)
- Standalone IGV HTML visualization of significant compartments (e.g., ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html)

## How to apply

First run dcHiC's compartment step: distance normalization (observed/expected) to a correlation matrix, parallelized partial SVD over the Filebacked Big Matrix to obtain PC1, then quantile normalization of compartment scores across all ESC/NPC maps so scores are comparable. For each 100 kb bin, compute the Mahalanobis distance across the per-replicate quantile-normalized scores — this multivariate z-score measures how much a bin is an outlier with respect to the compartment score distribution, capturing both compartment flips and within-compartment changes. Assign p-values by comparing the Mahalanobis distance to the chi-square distribution and correct with IHW (RefineY_Rconf90_FDR10 settings) or FDR, calling bins significant at FDR < 0.1. The same multivariate outlier testing is reused downstream per retained interaction to identify differential loops anchored in differential compartments.

## Related tools

- **dcHiC** (Implements the Mahalanobis-distance outlier test on quantile-normalized compartment scores, chi-square significance assignment, and IHW/FDR correction) — https://github.com/ay-lab/dcHiC
- **Fit-Hi-C** (Supplies per-replicate significant chromatin interaction calls (FDR < 0.05) whose anchors are filtered against differential compartments before per-interaction Mahalanobis testing) — https://github.com/ay-lab/fithic
- **HOMER** (Independent differential compartment scoring used to assess overlap/concordance with dcHiC's Mahalanobis-based calls (1355 of 3042 bins overlap, ±1 bin slack))

## Examples

```
bedtools pairtobed -a loops.bedpe -b dchic_diff_compartments_RefineY_Rconf90_FDR10.bed > anchored_loops.bedpe
```

## Evaluation signals

- ESC vs NPC pairwise comparison at 100 kb yields ~1981 significant bins at FDR < 0.1 under RefineY_Rconf90_FDR10 settings
- Flip-fraction annotation of significant bins totals ~74% compartment flips (approximately 30% A->B and 44% B->A)
- 1355 of HOMER's 3042 significant differential bins (FDR < 0.05) overlap dcHiC's calls within ±1 bin
- Adjusted p-values are present in the output table and significant bins include the reported ESC-specific (Dppa2/4) and NPC/CN-specific (Ephb1) regions
- Every bin with a called significance has a finite Mahalanobis distance and chi-square-derived p-value computed after quantile normalization across all replicate maps

## Limitations

- Compartment analysis from single-cell Hi-C is limited by contact-map sparsity, requiring pseudo-bulk aggregation of at least ~80-100 cells per condition
- Standard PCA-based compartment analysis handles one map at a time; cross-map comparability depends entirely on the quantile normalization step preceding the Mahalanobis test
- Validation in the paper was limited to human and mouse Hi-C data, though the method is presented as readily applicable to Hi-C variants

## Evidence

- [intro] Mahalanobis as multivariate outlier test: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Quantile normalization prerequisite: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Chi-square and multiple-testing correction: [evidence span withheld — anchor status: not_found]
- [readme] dcHiC scope across any number of cell lines: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
- [intro] Downstream differential interactions use case: "including standalone IGV browser37 visualization of results, detection of differential interactions involving significant differential compartments, time-series clustering of compartment scores, and"
