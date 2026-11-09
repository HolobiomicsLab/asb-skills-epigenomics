---
name: mahalanobis-distance-outlier-detection
description: Use when you must call statistically significant compartment differences
  between two or more Hi-C contact maps — including changes within the same A or B
  compartment type that standard PCA-on-each-map analysis cannot detect.
when_to_use_negative:
- You only need to identify compartments (A/B) within a single contact map — standard
  PCA per map suffices; this skill addresses between-map differential comparison
- Single-cell Hi-C maps are too sparse to yield reliable compartment scores and cannot
  be pseudo-bulked to ~100 cells per condition
- You need differential interaction (loop) calling rather than compartment-level differences
  — use Fit-Hi-C anchored on the significant compartments instead
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3957
tools:
- name: dcHiC
  role: 'Implements step-2 differential compartment calling: quantile normalization,
    MCD covariance with outlier removal, Mahalanobis distance, chi-square p-values,
    and IHW/FDR correction'
  repo: https://github.com/ay-lab/dcHiC
- name: IGV
  role: Standalone dynamic web-browser visualization of the significant differential
    compartment results
- name: Fit-Hi-C
  role: Downstream detection of differential loops anchored in the significant differential
    compartments identified by this skill
  repo: https://github.com/ay-lab/fithic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:4d531835a15130f3a479b1e0dcf090aa26e2d9733f499bcd64f9d94e5695db0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mahalanobis-distance-outlier-detection

## Summary

Statistical skill for flagging genomic bins whose compartment scores deviate significantly from the multivariate distribution across Hi-C datasets: dcHiC computes a per-bin Mahalanobis distance from quantile-normalized PC1/PC2 scores, then assigns chi-square p-values refined by robust covariance estimation.

## When to use

Use when you must call statistically significant compartment differences between two or more Hi-C contact maps — including changes within the same A or B compartment type that standard PCA-on-each-map analysis cannot detect.

## When NOT to use

- You only need to identify compartments (A/B) within a single contact map — standard PCA per map suffices; this skill addresses between-map differential comparison
- Single-cell Hi-C maps are too sparse to yield reliable compartment scores and cannot be pseudo-bulked to ~100 cells per condition
- You need differential interaction (loop) calling rather than compartment-level differences — use Fit-Hi-C anchored on the significant compartments instead

## Inputs

- Per-map compartment score matrices (PC1/PC2 from dcHiC step-1 partial SVD) in a binned score matrix across input Hi-C datasets
- Two or more normalized Hi-C contact maps, with or without replicates (resolutions up to 10 kb)
- Pseudo-bulk single-cell Hi-C profiles (as few as ~80–100 cells per condition)

## Outputs

- Per-genomic-bin table with Mahalanobis distance, raw chi-square p-value, and adjusted p-value (IHW or BH FDR)
- Set of significant differential compartment bins passing the FDR threshold (e.g., FDR 10%)

## How to apply

Load the per-map compartment score matrices (PC1/PC2 scores from dcHiC's step-1 partial SVD over a Filebacked Big Matrix) into a binned score matrix across all input Hi-C datasets, then apply quantile normalization so scores are comparable across maps/replicates. Fit a robust covariance estimate (Minimum Covariance Determinant, MCD), performing two passes of chi-square-based outlier removal to stabilize the covariance before final estimation. For each genomic bin, compute the weighted-center Mahalanobis distance — a multivariate z-score measuring how far the bin lies from the compartment score distribution center — and assign a chi-square p-value. Correct for multiple testing with IHW using per-replicate variance as the covariate (or Benjamini-Hochberg FDR); dcHiC's published ESC/NPC outputs use the RefineY_Rconf90_FDR10 (10% FDR) criterion.

## Related tools

- **dcHiC** (Implements step-2 differential compartment calling: quantile normalization, MCD covariance with outlier removal, Mahalanobis distance, chi-square p-values, and IHW/FDR correction) — https://github.com/ay-lab/dcHiC
- **IGV** (Standalone dynamic web-browser visualization of the significant differential compartment results)
- **Fit-Hi-C** (Downstream detection of differential loops anchored in the significant differential compartments identified by this skill) — https://github.com/ay-lab/fithic

## Examples

```
dchic --mode diff_compartment --input esc.npz,npc.npz --resolution 100000 --fdr 0.1 --out ESC_NPC_100Kb.RefineY_Rconf90_FDR10
```

## Evaluation signals

- Emitted table contains one row per genomic bin with columns for Mahalanobis distance, raw p-value, and adjusted p-value
- Significant bins match the FDR 10% criterion used in dcHiC's published ESC/NPC outputs (RefineY_Rconf90_FDR10 settings)
- Quantile normalization yields comparable compartment score distributions across all input maps/replicates before distance computation
- Covariance estimate is stable after two passes of chi-square-based outlier removal (robust MCD fit)
- Called differential compartments reproduce orthogonally validated changes (e.g., dynamically regulated genes, chromatin states, replication timing, lamin association) in the benchmark collections (mouse neural n=3, hematopoiesis n=10, LCLs n=20)

## Limitations

- Compartment identification from single-cell Hi-C maps remains limited by contact-map sparsity, even at coarse resolution; dcHiC requires pseudo-bulking (as low as ~80–100 cells per condition)
- Analysis demonstrated only on human and mouse Hi-C data in the paper, although the method is claimed applicable to Hi-C variants
- Comparison across thousands of cells and tens of conditions/clusters is deferred to future extensions
- Mahalanobis distance detects score-distribution outliers, including within-compartment (A-vs-A or B-vs-B) differences that should be interpreted alongside compartment identity

## Evidence

- [intro] dcHiC uses quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates and a multivariate distance measure to estimate statistical significance of compartment differences: [evidence span withheld — anchor status: not_found]
- [intro] Mahalanobis distance definition and testing: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [methods] Published output naming / FDR criterion: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
- [abstract] dcHiC role and multivariate measure: "Here, we introduce dcHiC, which utilizes a multivariate distance measure to identify significant changes in compartmentalization among multiple contact maps"
- [discussion] dcHiC scope for pseudo-bulk scHi-C: "dcHiC is able to call compartments and perform differential analysis from pseudo-bulk profiles of as low as 80 cells per condition"
