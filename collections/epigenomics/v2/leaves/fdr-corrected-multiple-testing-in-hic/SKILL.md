---
name: fdr-corrected-multiple-testing-in-hic
description: Use when calling significant differential compartments across two or
  more Hi-C contact maps and thousands of genomic bins must be tested, so per-bin
  chi-square p-values need multiple-testing correction at a target FDR (e.g. FDR10).
when_to_use_negative:
- The input is a single Hi-C map with no comparison condition — standard PCA-based
  compartment calling, not differential significance testing, is appropriate
- Only a small predefined set of loci is tested — per-locus p-values may not need
  genome-wide FDR correction
- The goal is differential interaction (loop) detection — Fit-Hi-C, not the compartment-level
  chi-square/FDR step, is the appropriate statistical framework
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dcHiC
  role: Computes Mahalanobis distances per genomic bin, assigns chi-square significance,
    and applies IHW or FDR correction to call significant differential compartments
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:74aa42466f5a8a03842fba94cfeeb06a9eabf5c7612f2107bbbc2accf8022e87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# FDR-corrected multiple testing in Hi-C differential compartment analysis

## Summary

dcHiC assigns per-bin statistical significance to differential compartment outliers by computing a Mahalanobis distance over quantile-normalized compartment scores, converting it to a chi-square p-value, and applying multiple-testing correction (FDR or IHW), e.g. the FDR10 setting used in the ESC-vs-NPC 100 kb analysis.

## When to use

Use when calling significant differential compartments across two or more Hi-C contact maps and thousands of genomic bins must be tested, so per-bin chi-square p-values need multiple-testing correction at a target FDR (e.g. FDR10).

## When NOT to use

- The input is a single Hi-C map with no comparison condition — standard PCA-based compartment calling, not differential significance testing, is appropriate
- Only a small predefined set of loci is tested — per-locus p-values may not need genome-wide FDR correction
- The goal is differential interaction (loop) detection — Fit-Hi-C, not the compartment-level chi-square/FDR step, is the appropriate statistical framework

## Inputs

- Quantile-normalized compartment scores (PC1/PC2) per genomic bin for each Hi-C contact map/replicate, e.g. ESC and NPC maps at 100 kb resolution
- Chosen multiple-testing correction setting (FDR10, as encoded in the RefineY_Rconf90_FDR10 run configuration)

## Outputs

- Per-bin chi-square p-values and FDR-adjusted significance calls for differential compartments
- Set of significant differential compartment bins (e.g. 1981 bins at full ESC-vs-NPC depth) for downstream recall or enrichment analyses

## How to apply

After dcHiC computes principal components and quantile-normalizes compartment scores across all input Hi-C maps/replicates, calculate for each genomic bin the Mahalanobis distance — a multivariate z-score measuring how far the bin deviates from the compartment score distribution. Convert this distance to a p-value via the chi-square test and correct for testing thousands of bins genome-wide using FDR correction (or IHW). In the ESC-vs-NPC 100 kb-resolution pairwise analysis, the 'RefineY_Rconf90_FDR10' configuration encodes an FDR10 significance setting; significant bins are then extracted (e.g. the 1981 significant differential bins at full depth) for downstream comparison or recall computation. Rationale: without correction, the sheer number of tested bins would yield an unmanageable false-positive rate for compartment outlier calls.

## Related tools

- **dcHiC** (Computes Mahalanobis distances per genomic bin, assigns chi-square significance, and applies IHW or FDR correction to call significant differential compartments) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- Corrected calls must respect the configured FDR level (e.g. FDR10 in RefineY_Rconf90_FDR10) — spot-check that estimated FDR does not exceed the target
- P-values must come from a chi-square test of Mahalanobis distances computed after quantile normalization across all maps/replicates, not from raw PC scores
- In the ESC-vs-NPC 100 kb down-sampling benchmark, the full-depth run yields 1981 significant differential bins, and the 3v3 analysis (excluding the 230M-read ESC replicate) recovers 1906 of them
- Down-sampled runs at ≥40% sequencing depth should retain over 80% recall of the full-depth differential bin set if significance testing and correction were applied consistently
- The significance output must feed the pcOri/visualization track (e.g. ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri) with consistent bin coordinates

## Limitations

- Compartment differences called with FDR control can include within-compartment (within-A or within-B) changes that lack obvious A/B flips, requiring careful biological interpretation
- Results depend on replicate quality and depth — a low-depth replicate (e.g. the 230M-read ESC replicate) can be excluded, shrinking the tested design from 4v3 to 3v3
- Sparsity of single-cell Hi-C maps limits reliable per-bin significance testing, which is why pseudo-bulk aggregation of ≥100 (or as low as ~80) cells per condition is used

## Evidence

- [intro] Mahalanobis distance plus chi-square and multiple-testing correction: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Quantile normalization precedes the significance step: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [readme] dcHiC role grounded in README: "dcHiC is a tool for differential compartment analysis of Hi-C datasets"
