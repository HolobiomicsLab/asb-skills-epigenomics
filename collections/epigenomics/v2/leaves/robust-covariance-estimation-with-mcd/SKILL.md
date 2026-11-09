---
name: robust-covariance-estimation-with-mcd
description: Use when you have compartment scores (PC1/PC2) from multiple Hi-C or
  pseudo-bulk scHi-C maps and need to test which genomic bins have statistically significant
  compartmentalization differences without relying on an A/B compartment flip.
when_to_use_negative:
- You have only a single Hi-C contact map and no comparative compartment scores —
  MCD/Mahalanobis significance requires two or more maps/replicates to define a multivariate
  null distribution.
- Your goal is calling differential interactions/loops rather than differential compartments
  — use dcHiC's Fit-Hi-C-linked differential interaction module instead.
- Compartment scores have not been quantile-normalized across maps/replicates — raw
  PC scores from independent PCAs are not comparable and would distort the covariance
  estimate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: dcHiC
  role: 'Implements step-2 differential compartment calling: MCD covariance estimation
    with outlier removal, weighted-center Mahalanobis distance per bin, chi-square
    p-values, and IHW/FDR correction'
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:c197de16214e74048526d38239b6b5db0d5af06f4bf237f71a0d7b1975897453
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# robust-covariance-estimation-with-mcd

## Summary

dcHiC's step-2 differential compartment module uses a Minimum Covariance Determinant (MCD) robust covariance estimate, stabilized by two passes of chi-square-based outlier removal, to compute weighted-center Mahalanobis distances from quantile-normalized compartment scores across two or more Hi-C maps. This turns per-bin PC1/PC2 score vectors into chi-square p-values for compartment differences.

## When to use

Use when you have compartment scores (PC1/PC2) from multiple Hi-C or pseudo-bulk scHi-C maps and need to test which genomic bins have statistically significant compartmentalization differences without relying on an A/B compartment flip.

## When NOT to use

- You have only a single Hi-C contact map and no comparative compartment scores — MCD/Mahalanobis significance requires two or more maps/replicates to define a multivariate null distribution.
- Your goal is calling differential interactions/loops rather than differential compartments — use dcHiC's Fit-Hi-C-linked differential interaction module instead.
- Compartment scores have not been quantile-normalized across maps/replicates — raw PC scores from independent PCAs are not comparable and would distort the covariance estimate.

## Inputs

- Binned compartment score matrix (PC1/PC2 scores per genomic bin) from dcHiC step-1 partial SVD across all input Hi-C datasets
- Quantile-normalized compartment scores across two or more Hi-C maps/replicates
- Per-replicate variance values (used as IHW covariate)

## Outputs

- Per-bin Mahalanobis distance values (multivariate z-scores)
- Chi-square raw p-values per genomic bin
- Adjusted p-values (IHW or BH-FDR) per genomic bin, significant bins at FDR 10%

## How to apply

First quantile-normalize the per-bin compartment score matrix (from dcHiC's step-1 partial SVD on the Filebacked Big Matrix) so scores are comparable across maps/replicates. Then fit a robust MCD covariance estimate on the normalized score distribution, applying two passes of chi-square-based outlier removal so outliers do not inflate the null covariance. For each genomic bin compute the Mahalanobis distance to the weighted center of the compartment score distribution — a multivariate z-score — and assign a chi-square p-value. Finally correct for multiple testing with IHW using per-replicate variance as the covariate (or Benjamini-Hochberg FDR), and call bins significant at the FDR 10% criterion (as in the RefineY_Rconf90_FDR10 ESC/NPC outputs).

## Related tools

- **dcHiC** (Implements step-2 differential compartment calling: MCD covariance estimation with outlier removal, weighted-center Mahalanobis distance per bin, chi-square p-values, and IHW/FDR correction) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- The output table contains one row per genomic bin with columns for Mahalanobis distance, raw p-value, and adjusted p-value.
- MCD covariance is computed twice: after the first chi-square pass, outliers are removed and the covariance is refit before the final significance assessment — verify two-pass behavior.
- Bins flagged significant match the FDR 10% criterion, consistent with the RefineY_Rconf90_FDR10 settings in the published ESC/NPC outputs.
- Distinct significant bins are found even where compartments do not flip (within-A or within-B differences), which is dcHiC's stated advantage over standard PCA.
- Significant compartment differences should agree with published ESC-vs-NPC results, including the ~10% of differences in nucleolus-associated domains.

## Limitations

- Compartment identification from single-cell Hi-C maps is limited by sparsity, even at coarse resolution; pseudo-bulk aggregation (down to as few as 80–100 cells per condition) is used to mitigate this.
- The method's statistical power depends on covariance estimation stability; outlier bins must be removed iteratively or the null distribution is inflated.
- Comparisons across thousands of cells and tens of conditions are deferred to future extensions of dcHiC.

## Evidence

- [intro] Mahalanobis distance and chi-square significance with IHW/FDR: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Quantile normalization prerequisite: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Scope: two or more contact maps, changes without compartment flip: "a method that identifies statistically significant differences in compartmentalization among two or more contact maps, including"
- [readme] Tool scope from README: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
