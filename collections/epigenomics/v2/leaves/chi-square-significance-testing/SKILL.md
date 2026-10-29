---
name: chi-square-significance-testing
description: Use when compartment scores (PC1/PC2) from two or more Hi-C contact maps
  are quantile-normalized and you must call per-bin significant compartment differences,
  including changes without a compartment flip.
when_to_use_negative:
- You only have a single Hi-C contact map with no second condition to compare — standard
  PCA per-map compartment calling is the right scope (standard PCA is limited to analyzing
  each contact map individually, which is why dcHiC's distance step exists)
- Compartment scores have not been quantile-normalized across maps/replicates; applying
  the distance measure to non-comparable scores invalidates the covariance estimate
- You need differential interaction calls rather than compartment-level differences
  — Fit-Hi-C (which dcHiC integrates for loops anchored in significant differential
  compartments) is the relevant tool
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: dcHiC
  role: Implements quantile normalization of compartment scores, MCD covariance estimation
    with outlier removal, weighted-center Mahalanobis distance computation, chi-square
    p-value assignment, and IHW/BH multiple-testing correction in its step-2 differential
    compartment module
  repo: https://github.com/ay-lab/dcHiC
- name: Fit-Hi-C
  role: 'Downstream integration: identifies differential loops anchored in bins called
    as significant differential compartments by the chi-square/Mahalanobis step'
  repo: https://github.com/ay-lab/fithic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:d6165a3699f573da44b525abb47c70e1aaa5b2f51a243b242e1db4cf4ef392bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - dcHiC
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chi-square-significance-testing

## Summary

dcHiC's step-2 differential compartment procedure: compute a per-bin Mahalanobis distance on quantile-normalized PC1/PC2 compartment scores, assign chi-square p-values to flag outlier bins, and correct for multiple testing with IHW or Benjamini-Hochberg FDR.

## When to use

Use when compartment scores (PC1/PC2) from two or more Hi-C contact maps are quantile-normalized and you must call per-bin significant compartment differences, including changes without a compartment flip.

## When NOT to use

- You only have a single Hi-C contact map with no second condition to compare — standard PCA per-map compartment calling is the right scope (standard PCA is limited to analyzing each contact map individually, which is why dcHiC's distance step exists)
- Compartment scores have not been quantile-normalized across maps/replicates; applying the distance measure to non-comparable scores invalidates the covariance estimate
- You need differential interaction calls rather than compartment-level differences — Fit-Hi-C (which dcHiC integrates for loops anchored in significant differential compartments) is the relevant tool

## Inputs

- Binned compartment score matrix (PC1/PC2 scores from dcHiC step-1 partial SVD) across all input Hi-C datasets
- Quantile-normalized compartment scores per genomic bin for two or more Hi-C maps/replicates
- Optionally, per-replicate variance values to use as the IHW covariate

## Outputs

- Per-genomic-bin table with Mahalanobis distance, raw chi-square p-value, and adjusted p-value
- Set of bins passing the significance threshold (e.g., FDR 10%) labeling significant differential compartments

## How to apply

First normalize compartment scores with quantile normalization across all input Hi-C maps/replicates so they are comparable between conditions. Fit a robust (Minimum Covariance Determinant) covariance estimate on the normalized score distribution, performing two passes of chi-square-based outlier removal to stabilize the covariance. For each genomic bin, compute the weighted-center Mahalanobis distance — a multivariate z-score of how far the bin lies from the score distribution center — and assign a chi-square p-value from that distance. Correct for multiple testing using IHW with per-replicate variance as the covariate, or fall back to Benjamini-Hochberg FDR correction; dcHiC's published ESC/NPC outputs use an FDR 10% criterion (e.g., RefineY_Rconf90_FDR10 settings).

## Related tools

- **dcHiC** (Implements quantile normalization of compartment scores, MCD covariance estimation with outlier removal, weighted-center Mahalanobis distance computation, chi-square p-value assignment, and IHW/BH multiple-testing correction in its step-2 differential compartment module) — https://github.com/ay-lab/dcHiC
- **Fit-Hi-C** (Downstream integration: identifies differential loops anchored in bins called as significant differential compartments by the chi-square/Mahalanobis step) — https://github.com/ay-lab/fithic

## Evaluation signals

- Emitted table contains one row per genomic bin with columns for Mahalanobis distance, raw p-value, and adjusted p-value
- Bins passing the significance threshold match the FDR 10% criterion used by dcHiC (e.g., RefineY_Rconf90_FDR10 settings in published ESC/NPC outputs)
- Covariance estimation was performed twice: initial MCD fit, then recalculated after two passes of chi-square-based outlier removal
- Compartment scores entering the distance calculation are comparable across conditions (quantile-normalized), not raw per-map PC1/PC2 values
- Multiple-testing correction used IHW with per-replicate variance covariate or BH FDR, not uncorrected p-values

## Limitations

- Analysis in the published work was limited to human and mouse Hi-C data, though the method is stated to be readily applicable to Hi-C variants
- Genomic compartment identification from single-cell Hi-C maps is limited by sparsity, so dcHiC relies on pseudo-bulk aggregation (works from ~80-100 cells per condition)
- Compartment comparison across thousands of cells and tens of conditions is deferred to future extensions of dcHiC

## Evidence

- [intro] Per-bin Mahalanobis distance and chi-square significance: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Quantile normalization prerequisite: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] IHW/FDR correction step: [evidence span withheld — anchor status: not_found]
- [methods] Published FDR 10% threshold naming convention: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
- [readme] dcHiC scope of significance calling: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
