---
name: multiple-testing-fdr-correction
description: Use when dcHiC has produced one raw chi-square p-value per genomic bin
  (from Mahalanobis distance on quantile-normalized compartment scores) or per candidate
  interaction, and you must call significant differential compartments or interactions
  at a controlled adjusted-p threshold such as FDR 10%.
when_to_use_negative:
- You have a single hypothesis test or per-test alpha control is required (e.g., permutation-derived
  empirical p-values already calibrated) — family-wise or empirical calibration is
  a different problem than FDR control over genome-wide bin sets
- Input is a single Hi-C map with no replicates or comparison, since there is no multivariate
  Mahalanobis p-value distribution to correct
- The p-values come from a different compartment tool (e.g., HOMER differential scores)
  whose null distribution assumptions differ from dcHiC's chi-square-on-Mahalanobis-distance
  scheme
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Computes Mahalanobis-distance chi-square p-values per genomic bin and applies
    IHW (with per-replicate variance covariate) or Benjamini-Hochberg FDR correction
    to call significant differential compartments and differential interactions
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:187a2c58a04362e9a633cf75cebc056ce78b4218ceb714c9799be3953518a228
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multiple-testing-fdr-correction

## Summary

Adjusts per-bin chi-square p-values from dcHiC's Mahalanobis-distance differential compartment test (and p-values from its differential interaction module) for multiple testing using IHW with per-replicate variance as covariate, or Benjamini-Hochberg FDR, so that significant compartment differences are called at a controlled false discovery rate (e.g., FDR 10%).

## When to use

Use when dcHiC has produced one raw chi-square p-value per genomic bin (from Mahalanobis distance on quantile-normalized compartment scores) or per candidate interaction, and you must call significant differential compartments or interactions at a controlled adjusted-p threshold such as FDR 10%.

## When NOT to use

- You have a single hypothesis test or per-test alpha control is required (e.g., permutation-derived empirical p-values already calibrated) — family-wise or empirical calibration is a different problem than FDR control over genome-wide bin sets
- Input is a single Hi-C map with no replicates or comparison, since there is no multivariate Mahalanobis p-value distribution to correct
- The p-values come from a different compartment tool (e.g., HOMER differential scores) whose null distribution assumptions differ from dcHiC's chi-square-on-Mahalanobis-distance scheme

## Inputs

- Per-genomic-bin raw chi-square p-values from dcHiC's Mahalanobis distance calculation on quantile-normalized compartment scores
- Per-replicate variance of compartment scores (IHW covariate)
- Raw p-values from dcHiC's differential interaction (Mahalanobis outlier) test on candidate interactions from Fit-Hi-C, HiCCUPS, or Mustache

## Outputs

- Differential compartment table with one row per genomic bin containing Mahalanobis distance, raw p-value, and adjusted p-value, with bins passing the FDR threshold (e.g., FDR 10%) flagged as significant
- Differential interactions table (e.g., differential_interactions.tsv) with adjusted p-values per interaction

## How to apply

After dcHiC computes, for each genomic bin, the weighted-center Mahalanobis distance from robustly (MCD) estimated compartment-score covariance after two passes of chi-square-based outlier removal, convert the distance to a chi-square raw p-value. Then apply Independent Hypothesis Weighting (IHW) using per-replicate variance as the covariate — preferred because bins with noisy replicate compartment scores are penalized appropriately — or fall back to Benjamini-Hochberg FDR correction. The resulting adjusted p-values define significance settings such as the published ESC/NPC RefineY_Rconf90_FDR10 configuration (Refine.Y covariance, R-conf 90%, FDR 10% cutoff). The same correction logic applies downstream to p-values of differential interactions (from Fit-Hi-C/HiCCUPS/Mustache calls filtered for overlap with differential compartments) before writing the differential interactions table. Verify significance labels match the intended FDR threshold in downstream outputs (e.g., differential_interactions.tsv, IGV HTML views).

## Related tools

- **dcHiC** (Computes Mahalanobis-distance chi-square p-values per genomic bin and applies IHW (with per-replicate variance covariate) or Benjamini-Hochberg FDR correction to call significant differential compartments and differential interactions) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- Output table contains one row per genomic bin with columns for Mahalanobis distance, raw p-value, and adjusted p-value
- Bins flagged significant match the FDR 10% criterion used by dcHiC's published RefineY_Rconf90_FDR10 settings (ESC vs NPC at 100 kb)
- Significant differential interactions are anchored at reported ESC-specific (Dppa2/4) and NPC/CN-specific (Ephb1) differential compartment regions under RefineY_Rconf90_FDR10
- Adjusted p-values are monotonically consistent with raw p-values and all fall within [0,1], with the number of significant bins at FDR 10% not exceeding those at FDR < 0.05 raw cutoff comparisons in the expected direction
- When IHW is used, per-replicate variance covariate values are present and finite for all bins with tested p-values

## Limitations

- IHW requires a covariate (per-replicate variance); without replicate structure the user must fall back to Benjamini-Hochberg FDR, which can be less powerful
- Compartment analysis from single-cell Hi-C is hampered by sparsity even at coarse resolution, weakening the p-value distribution being corrected; pseudo-bulk aggregation from as few as ~80–100 cells per condition is recommended
- The corrected calls reflect within-A or within-B compartment strength differences as well as flips, so a significant adjusted p-value does not imply a compartment sign change

## Evidence

- [intro] dcHiC uses quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates and a multivariate distance measure to estimate statistical significance of compartment differences: [evidence span withheld — anchor status: not_found]
- [intro] for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [other] verify that the emitted table contains one row per genomic bin with columns for Mahalanobis distance, raw p-value, and adjusted p-value, and that bins passing the significance threshold match the FDR 10% criterion: [evidence span withheld — anchor status: not_found]
- [methods] ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
- [readme] Identification of differential loops anchored in significant differential compartments (using Fit-Hi-C): "Identification of differential loops anchored in significant differential compartments (using [Fit-Hi-C](https://github.com/ay-lab/fithic))"
