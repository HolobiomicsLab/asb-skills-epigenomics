---
name: hic-differential-interaction-testing
description: Use when comparing Hi-C datasets (e.g., ESC vs NPC) and you need loop-level
  differences, not just compartment-level calls, specifically for interactions anchored
  in bins flagged as significant differential compartments.
when_to_use_negative:
- You only need compartment-level A/B differences — use dcHiC's differential compartment
  analysis instead, since this skill is scoped to interactions anchored in differential
  compartments
- You have only a single Hi-C contact map (no condition comparison) — differential
  interaction testing requires two or more contact maps
- Your input is raw Hi-C contact matrices without prior significant interaction calls
  — Fit-Hi-C (or HiCCUPS/Mustache) loop calling must be run first
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: dcHiC
  role: Runs the Mahalanobis-distance multivariate outlier test per interaction across
    samples, chi-square p-value assignment, and IHW/FDR multiple-testing correction
    for differential interactions
  repo: https://github.com/ay-lab/dcHiC
- name: Fit-Hi-C
  role: Provides the per-replicate significant interaction calls (FDR < 0.05) that
    serve as the input interaction set to be filtered and tested
  repo: https://github.com/ay-lab/fithic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:fc6928f267cef0066b82719c9ef0c7710c0d0357841e729441114b9b596e21f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hic-differential-interaction-testing

## Summary

Identify chromatin interactions (significant contacts or loops) whose contact strength differs across conditions, by first filtering the interaction set for anchors overlapping significant differential compartments and then testing each retained interaction with a multivariate outlier statistic, as implemented in dcHiC's differential interaction feature.

## When to use

Use when comparing Hi-C datasets (e.g., ESC vs NPC) and you need loop-level differences, not just compartment-level calls, specifically for interactions anchored in bins flagged as significant differential compartments.

## When NOT to use

- You only need compartment-level A/B differences — use dcHiC's differential compartment analysis instead, since this skill is scoped to interactions anchored in differential compartments
- You have only a single Hi-C contact map (no condition comparison) — differential interaction testing requires two or more contact maps
- Your input is raw Hi-C contact matrices without prior significant interaction calls — Fit-Hi-C (or HiCCUPS/Mustache) loop calling must be run first

## Inputs

- Per-replicate significant interaction calls from Fit-Hi-C at FDR < 0.05 (or loop calls from HiCCUPS/Mustache)
- dcHiC differential compartment calls (e.g., RefineY_Rconf90_FDR10) at 100 kb resolution for the compared conditions
- Distance-normalized Hi-C contact maps (observed/expected) for each sample

## Outputs

- Differential interactions table with adjusted p-values (differential_interactions.tsv)
- Per-interaction log2(Observed/Expected) contact strength values across samples
- Mahalanobis-distance test statistics with chi-square p-values and multiple-testing corrections

## How to apply

Start from per-replicate significant interaction calls generated with Fit-Hi-C (FDR < 0.05) or loop calls from HiCCUPS/Mustache, together with dcHiC differential compartment calls (e.g., RefineY_Rconf90_FDR10 at 100 kb resolution). Retain only interactions whose anchor intervals overlap bins flagged as significant differential compartments (e.g., with bedtools pairtobed). For each retained interaction, compute log2(Observed/Expected) contact strength per sample from the distance-normalized maps, then apply dcHiC's Mahalanobis-distance multivariate outlier test per interaction, assigning p-values via the chi-square distribution. Correct p-values for multiple testing (IHW or FDR) and write the differential interactions table with adjusted p-values (e.g., differential_interactions.tsv).

## Related tools

- **dcHiC** (Runs the Mahalanobis-distance multivariate outlier test per interaction across samples, chi-square p-value assignment, and IHW/FDR multiple-testing correction for differential interactions) — https://github.com/ay-lab/dcHiC
- **Fit-Hi-C** (Provides the per-replicate significant interaction calls (FDR < 0.05) that serve as the input interaction set to be filtered and tested) — https://github.com/ay-lab/fithic

## Examples

```
bedtools pairtobed -a fithic_significant_interactions.txt -b dchic_differential_compartments_RefineY_Rconf90_FDR10.bed > differential_interaction_candidates.txt
```

## Evaluation signals

- Output table contains differential interactions anchored at the reported ESC-specific (Dppa2/4) and NPC/CN-specific (Ephb1) differential compartment regions under RefineY_Rconf90_FDR10 settings
- Every retained interaction has at least one anchor interval overlapping a bin flagged as a significant differential compartment (verifiable by re-running the pairtobed filter)
- Adjusted p-values are present in the output and are consistent with IHW or FDR correction of chi-square p-values derived from Mahalanobis distances
- All input interactions should originate from significant calls meeting the FDR < 0.05 threshold from Fit-Hi-C (or HiCCUPS/Mustache loops)

## Limitations

- Only interactions anchored in significant differential compartments are tested — interactions changing between conditions but outside differential compartments are excluded by the filter
- dcHiC analysis requires multiple contact maps (two or more conditions); standard PCA-based compartment comparison is limited to analyzing each Hi-C contact map individually
- Quality of differential interaction calls depends on upstream significant interaction calls (Fit-Hi-C/HiCCUPS/Mustache) and on the differential compartment significance settings (e.g., RefineY_Rconf90_FDR10)

## Evidence

- [abstract] dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering: "dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering"
- [readme] dcHiC identifies differential loops anchored in significant differential compartments using Fit-Hi-C calls: "Identification of differential loops anchored in significant differential compartments (using [Fit-Hi-C](https://github.com/ay-lab/fithic))"
- [intro] dcHiC calculates the Mahalanobis distance per bin, assigns significance via chi-square test, and applies IHW or FDR correction: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] dcHiC's downstream feature: detection of differential interactions involving significant differential compartments: "including standalone IGV browser37 visualization of results, detection of differential interactions involving significant differential compartments, time-series clustering of compartment scores, and"
