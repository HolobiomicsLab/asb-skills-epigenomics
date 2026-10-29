---
name: genomic-interval-overlap-filtering
description: Use when you have per-replicate significant interaction/loop calls (e.g.,
  FitHiC at FDR < 0.05, HiCCUPS or Mustache loops) and a list of significant differential
  compartment bins (e.g., RefineY_Rconf90_FDR10), and need only the interactions anchored
  in those bins.
when_to_use_negative:
- You only need compartment-level differences without interaction filtering — run
  dcHiC's differential compartment feature directly instead of subsetting loops.
- Interaction calls and compartment calls are at mismatched resolutions (compartment
  calls were made at 100 kb but loop anchors at 5 kb bins) — bin harmonization must
  precede the overlap step.
- The features to filter are not interval-paired data (e.g., single intervals rather
  than BEDPE pairs) — a plain bedtools intersect, not pairtobed, is appropriate.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: pairtobed subcommand retains only interactions whose anchor intervals overlap
    bins flagged as significant differential compartments
- name: dcHiC
  role: provides the differential compartment calls used as the overlap filter set
    and consumes the filtered interactions for its differential interaction (Mahalanobis-distance)
    test
  repo: https://github.com/ay-lab/dcHiC
- name: Fit-Hi-C
  role: generates the per-replicate significant interaction calls that are filtered
    by overlap with differential compartments
  repo: https://github.com/ay-lab/fithic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:79021e2921ac24de68278e710be6fbe3041638c266086435c5ec5d78be4283c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - bedtools
  - dcHiC
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-interval-overlap-filtering

## Summary

Filters a set of genomic features (e.g., significant chromatin interactions or loops) by retaining only those whose anchor intervals overlap a second feature set (e.g., significant differential compartments called by dcHiC). In the dcHiC ESC vs NPC workflow this isolates differential interactions anchored in differential compartments at 100 kb resolution.

## When to use

Use when you have per-replicate significant interaction/loop calls (e.g., FitHiC at FDR < 0.05, HiCCUPS or Mustache loops) and a list of significant differential compartment bins (e.g., RefineY_Rconf90_FDR10), and need only the interactions anchored in those bins.

## When NOT to use

- You only need compartment-level differences without interaction filtering — run dcHiC's differential compartment feature directly instead of subsetting loops.
- Interaction calls and compartment calls are at mismatched resolutions (compartment calls were made at 100 kb but loop anchors at 5 kb bins) — bin harmonization must precede the overlap step.
- The features to filter are not interval-paired data (e.g., single intervals rather than BEDPE pairs) — a plain bedtools intersect, not pairtobed, is appropriate.

## Inputs

- Per-replicate significant interaction calls from FitHiC (FDR < 0.05), in BEDPE/pair format
- Significant chromatin loop calls from HiCCUPS or Mustache (alternative interaction source)
- dcHiC differential compartment calls at the RefineY_Rconf90_FDR10 significance setting, 100 kb resolution (BED of bins)

## Outputs

- Filtered BEDPE of interactions whose anchor intervals overlap significant differential compartment bins
- Differential interactions table (differential_interactions.tsv) with adjusted p-values after the dcHiC Mahalanobis-distance test and IHW/FDR correction

## How to apply

Load the significant interaction calls (FitHiC FDR < 0.05 per replicate, or HiCCUPS/Mustache loops) as anchor intervals and the dcHiC differential compartment calls (RefineY, Rconf90, FDR10 settings at 100 kb for ESC vs NPC) as the filter set. Use bedtools pairtobed to retain only pairs whose anchor intervals overlap bins flagged as significant differential compartments. This filtered set is then the input to dcHiC's downstream differential interaction test, where log2(Observed/Expected) contact strength is computed per sample from distance-normalized maps and interactions are scored with the Mahalanobis-distance multivariate outlier test, p-values assigned via the chi-square distribution, and corrected with IHW or FDR. The rationale is that dcHiC's differential interaction feature only tests loops that are anchored in significant differential compartments, so this overlap step defines which candidate interactions enter the statistical test.

## Related tools

- **bedtools** (pairtobed subcommand retains only interactions whose anchor intervals overlap bins flagged as significant differential compartments)
- **dcHiC** (provides the differential compartment calls used as the overlap filter set and consumes the filtered interactions for its differential interaction (Mahalanobis-distance) test) — https://github.com/ay-lab/dcHiC
- **Fit-Hi-C** (generates the per-replicate significant interaction calls that are filtered by overlap with differential compartments) — https://github.com/ay-lab/fithic

## Examples

```
bedtools pairtobed -a fithic_significant_pairs.bedpe -b ESC_NPC_100Kb.RefineY_Rconf90_FDR10_differential_bins.bed > differential_anchored_pairs.bedpe
```

## Evaluation signals

- Output BEDPE is a strict subset of the input FitHiC/loop calls — no pairs added, only removed by the overlap filter.
- Every retained interaction has at least one anchor interval overlapping a bin flagged significant under RefineY_Rconf90_FDR10 settings.
- Validation: differential_interactions.tsv contains differential interactions anchored at the reported ESC-specific (Dppa2/4) and NPC/CN-specific (Ephb1) differential compartment regions.
- Retained pairs carry both the FitHiC FDR < 0.05 significance from the source calls and dcHiC-adjusted p-values (IHW or FDR corrected) after testing.
- Count of retained pairs is non-trivially smaller than the full interaction call set, confirming the filter discriminates on compartment overlap.

## Limitations

- dcHiC only tests interactions involving significant differential compartments; interactions between non-differential bins are never surfaced by this pipeline.
- The filter inherits the significance settings of the compartment calls (e.g., RefineY_Rconf90_FDR10); different settings change which anchors pass.
- Resolution mismatch between loop anchor bins and 100 kb compartment bins can cause false negative overlaps unless intervals are harmonized.
- Per the article, compartment identification from single-cell Hi-C maps is still limited by sparsity, affecting the compartment calls used as the filter set.

## Evidence

- [full_text] dcHiC's differential interaction feature takes as input a set of significant chromatin interactions (e.g., from Fit-Hi-C) or chromatin loops (e.g., from HiCCUPS or Mustache), filters them for their overlap with differential compartments, and tests them for their difference across the compared conditions.: [evidence span withheld — anchor status: not_found]
- [readme] README of ay-lab/dcHiC lists differential loops as a feature.: "Identification of differential loops anchored in significant differential compartments (using [Fit-Hi-C](https://github.com/ay-lab/fithic))"
- [intro] Article intro lists downstream features including the interaction detection this filtering serves.: "including standalone IGV browser37 visualization of results, detection of differential interactions involving significant differential compartments, time-series clustering of compartment scores, and"
- [intro] dcHiC significance testing for the interactions retained by the overlap filter.: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [methods] Methods artifact showing the RefineY_Rconf90_FDR10 significance setting used for compartment calls in the ESC vs NPC comparison.: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
