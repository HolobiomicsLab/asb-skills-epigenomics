---
name: hic-compartment-score-quantile-normalization-across-replicates
description: Use when comparing Hi-C compartment scores (e.g., PC1 at 100 kb bins)
  across two or more conditions or replicates (e.g., mouse ESC vs NPC) so scores are
  on a common scale before significance testing.
when_to_use_negative:
- A single Hi-C map is being analyzed with no cross-map or cross-replicate comparison
  — plain PCA per map suffices and quantile normalization across maps is meaningless
- Single-cell Hi-C maps are used without pseudo-bulk aggregation — sparsity of single-cell
  contact maps makes per-cell compartment scores unreliable (dcHiC instead uses pseudo-bulk
  profiles from ~100 cells per condition)
- The goal is differential loop/contact detection rather than compartmentalization
  — Fit-Hi-C-based differential interaction analysis is the appropriate tool
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Computes principal components (parallelized partial SVD over Filebacked Big
    Matrix) and applies quantile normalization of compartment scores across all input
    maps, then differential testing
  repo: https://github.com/ay-lab/dcHiC
- name: IGV
  role: Standalone dynamic IGV web browser view for visualizing the normalized compartment
    scores and differential calls
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Independent differential compartment scorer used for orthogonal comparison
    of quantile-normalized dcHiC calls (e.g., 3042 differential bins at FDR<0.05,
    1355 overlapping dcHiC calls)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:cc1cb10517e641ed9db14decfb95fa5a6bf2f4e436709e45c4fc0d0bfb911ec8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hic-compartment-score-quantile-normalization-across-replicates

## Summary

dcHiC's two-step procedure for making compartment scores comparable across multiple Hi-C maps: after computing PC1 scores via distance-normalized correlation matrices and a parallelized partial SVD (Filebacked Big Matrix), quantile normalization is applied across all input maps/replicates so that downstream multivariate differential testing (Mahalanobis distance) is valid.

## When to use

Use when comparing Hi-C compartment scores (e.g., PC1 at 100 kb bins) across two or more conditions or replicates (e.g., mouse ESC vs NPC) so scores are on a common scale before significance testing.

## When NOT to use

- A single Hi-C map is being analyzed with no cross-map or cross-replicate comparison — plain PCA per map suffices and quantile normalization across maps is meaningless
- Single-cell Hi-C maps are used without pseudo-bulk aggregation — sparsity of single-cell contact maps makes per-cell compartment scores unreliable (dcHiC instead uses pseudo-bulk profiles from ~100 cells per condition)
- The goal is differential loop/contact detection rather than compartmentalization — Fit-Hi-C-based differential interaction analysis is the appropriate tool

## Inputs

- Hi-C contact maps for two or more conditions, including replicates (e.g., mouse ESC and NPC maps at 100 kb resolution)
- Chromosome-arm/side partition of each contact map for distance normalization to a correlation matrix
- Choice of principal component (PC1 or PC2, sign-confirmed per condition) representing A/B compartments

## Outputs

- Quantile-normalized per-bin compartment scores for every map/replicate (comparable across conditions)
- Per-bin Mahalanobis distances and chi-square p-values with IHW-adjusted FDR significance calls (e.g., RefineY_Rconf90_FDR10)
- Per-bin A/B compartment assignments and flip annotations (A->B, B->A, within-compartment) written to results_table.tsv

## How to apply

Load the Hi-C contact maps (e.g., ESC and NPC, including replicates, binned at 100 kb) and for each chromosome perform distance normalization (observed/expected), transform to a correlation matrix, and run dcHiC's parallelized partial SVD using the Filebacked Big Matrix (FBM) structure to obtain PC1 (or PC2) compartment scores. Then apply quantile normalization across all input maps/replicates so scores are directly comparable — this is the prerequisite for dcHiC's second step, where each bin's Mahalanobis distance (a multivariate z-score) is assigned a chi-square p-value and IHW/FDR correction calls significant differential compartments. In the ESC vs NPC case study this pipeline used the RefineY_Rconf90_FDR10 settings and produced per-bin scores feeding the FDR<0.1 differential calls. The rationale is that raw PCA scores are not calibrated across maps, so within-compartment (A-to-A or B-to-B) changes would be missed without quantile-normalized, comparable scores.

## Related tools

- **dcHiC** (Computes principal components (parallelized partial SVD over Filebacked Big Matrix) and applies quantile normalization of compartment scores across all input maps, then differential testing) — https://github.com/ay-lab/dcHiC
- **IGV** (Standalone dynamic IGV web browser view for visualizing the normalized compartment scores and differential calls) — https://github.com/ay-lab/dcHiC
- **HOMER** (Independent differential compartment scorer used for orthogonal comparison of quantile-normalized dcHiC calls (e.g., 3042 differential bins at FDR<0.05, 1355 overlapping dcHiC calls))

## Evaluation signals

- After quantile normalization, compartment score distributions (quantiles) are comparable across all ESC/NPC maps and replicates — no single map dominates the scale
- Downstream calls reproduce paper-level invariants: ~1981 significant 100 kb bins at FDR<0.1 with RefineY_Rconf90_FDR10 settings
- Flip fractions among significant bins total ~74% (A->B ~30%, B->A ~44%), with the remainder being within-compartment changes
- Overlap check against an orthogonal method: 1,355 of HOMER's 3,042 differential bins (FDR<0.05) overlap dcHiC's calls (±1 bin slack)
- Results (scores, significance calls, flip annotations, HOMER-overlap flags) are written to results_table.tsv and visualizable in the dcHiC-generated HTML/IGV view (e.g., ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html)

## Limitations

- Compartment identification from sparse single-cell contact maps remains a major issue; pseudo-bulk aggregation is required (dcHiC needs roughly 100 cells per condition)
- Analysis in the paper was limited to human and mouse Hi-C data, though the method is applicable to other Hi-C variants
- Standard PCA-based compartment analysis is limited to one map at a time; without cross-map quantile normalization, within-compartment differences (A-to-A, B-to-B) would be ignored

## Evidence

- [intro] dcHiC calculates principal components followed by quantile normalization of compartment scores across all input Hi-C data: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [intro] Quantile normalization yields comparable compartment scores across maps/replicates: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Partial SVD over Filebacked Big Matrix for compartment scores: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Mahalanobis distance with chi-square significance and IHW/FDR correction per bin: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] dcHiC performs compartment analysis on maps with or without replicates at up to 10 kb resolution: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb and for pseudo-bulk Hi-C profiles from as few as 100 cells per condition"
- [readme] dcHiC is a tool for differential compartment analysis of Hi-C datasets: "dcHiC is a tool for differential compartment analysis of Hi-C datasets. It features many capabilities, including:"
