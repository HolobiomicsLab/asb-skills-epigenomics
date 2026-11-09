---
name: cohesin-depletion-loop-loss-quantification
description: Use when validating a supervised chromatin loop caller on a perturbation
  system where loops are expected to disappear, such as a cohesin (RAD21-AID) auxin-depletion
  Hi-C experiment, to estimate the model's false discovery rate.
when_to_use_negative:
- The input maps are not a biological perturbation in which loops are expected to
  disappear (e.g. two untreated cell types) — residual loops then do not straightforwardly
  represent false positives.
- The pre-trained model was not trained on raw contact values (a normalized weight
  was used in training) — in that case --clr-weight-name must match the model's training
  normalization instead of 'raw'.
- The goal is de novo loop discovery rather than model validation; use peakachu train
  with a positive bedpe training set instead.
tools:
- name: Peakachu
  role: Random Forest loop detector whose CTCF ChIA-PET-trained models score pixels
    genome-wide (score_genome) and pool significant non-redundant loops (pool) in
    both auxin-treated and untreated maps
  repo: https://github.com/tariks/peakachu
- name: cooler
  role: Reads the .mcool/.cool RAD21-AID Hi-C contact maps at 10 kb resolution for
    scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:bdff2d0f3bbfc65535a79e21d2c709da2b46781d74a562f44405527b99ebf13f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cohesin-depletion-loop-loss-quantification

## Summary

Estimates the false discovery rate of the Peakachu chromatin loop detector by scoring matched auxin-treated (cohesin-depleted) and untreated HCT-116 RAD21-AID Hi-C maps with the same CTCF ChIA-PET-trained Random Forest model, using the near-total loss of loops upon cohesin depletion (19 vs 11,814 loops) to derive an FDR of ~0.2%.

## When to use

Use when validating a supervised chromatin loop caller on a perturbation system where loops are expected to disappear, such as a cohesin (RAD21-AID) auxin-depletion Hi-C experiment, to estimate the model's false discovery rate.

## When NOT to use

- The input maps are not a biological perturbation in which loops are expected to disappear (e.g. two untreated cell types) — residual loops then do not straightforwardly represent false positives.
- The pre-trained model was not trained on raw contact values (a normalized weight was used in training) — in that case --clr-weight-name must match the model's training normalization instead of 'raw'.
- The goal is de novo loop discovery rather than model validation; use peakachu train with a positive bedpe training set instead.

## Inputs

- Auxin-treated HCT-116 RAD21-AID Hi-C contact map (.mcool or .cool file, 10 kb resolution)
- Untreated HCT-116 RAD21-AID Hi-C contact map (.mcool or .cool file, 10 kb resolution, matched sequencing depth)
- Pre-trained CTCF ChIA-PET Peakachu Random Forest model pickle (CTCF-ChIAPET-peakachu-pretrained.*.10kb.pkl)

## Outputs

- Per-pixel loop probability bedpe files for each condition (score_genome output)
- Pooled non-redundant loop centroid lists per condition (peakachu pool output, e.g. at probability threshold 0.95)
- Loop counts per condition (e.g. 19 auxin-treated vs 11,814 untreated) and derived FDR estimate (~0.2%)

## How to apply

Load the auxin-treated and untreated HCT-116 RAD21-AID Hi-C contact maps as .mcool/.cool files at 10 kb resolution with cooler. Run 'peakachu score_genome -r 10000' on each map using the CTCF ChIA-PET pre-trained Random Forest model (CTCF-ChIAPET-peakachu-pretrained.*.10kb.pkl), setting --clr-weight-name to raw because the CTCF ChIA-PET models were trained on raw contact values. Pool the per-pixel probability bedpe outputs with 'peakachu pool' using a high probability threshold (e.g. -t 0.95) to select the most significant non-redundant loops. Count pooled loops in both conditions; the residual loop count in the cohesin-depleted map relative to the untreated map (19/11,814) estimates the FDR, since cohesin depletion should abolish loop extrusion-dependent CTCF loops.

## Related tools

- **Peakachu** (Random Forest loop detector whose CTCF ChIA-PET-trained models score pixels genome-wide (score_genome) and pool significant non-redundant loops (pool) in both auxin-treated and untreated maps) — https://github.com/tariks/peakachu
- **cooler** (Reads the .mcool/.cool RAD21-AID Hi-C contact maps at 10 kb resolution for scoring)

## Examples

```
peakachu score_genome -r 10000 -p auxin.mcool::resolutions/10000 --clr-weight-name raw -m CTCCF-ChIAPET-peakachu-pretrained.*.10kb.pkl -O auxin_scores.tsv
```

## Evaluation signals

- Untreated map at matched sequencing depth yields ~11,814 pooled loops, matching the reported count.
- Auxin-treated (cohesin-depleted) map yields ~19 pooled loops, matching the reported count.
- The derived FDR, 19/11,814, equals ~0.2%.
- Peakachu 'pool' is run with the same non-redundant selection threshold (e.g. -t 0.95) in both conditions so counts are comparable.
- The --clr-weight-name is set to 'raw' consistently for both conditions, matching the raw-contact training of the CTCF ChIA-PET model.

## Limitations

- The FDR estimate assumes all true cohesin-dependent loops disappear upon auxin treatment; any residual real loops in the auxin map would inflate the apparent false discovery count.
- The estimate applies to the CTCF ChIA-PET-trained model at 10 kb resolution with the specific pool probability threshold; other models or cutoffs (e.g. 0.97 for CTCF, 0.92 for H3K27ac in the paper) need separate calibration.
- Requires the two conditions to be at similar sequencing depth for the loop-count ratio to be meaningful.

## Evidence

- [methods] The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [readme] Specify it to "raw" if you want to use the raw signals.: "Specify it to "raw" if you want to use the raw signals."
- [discussion] Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set.: "Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set."
- [readme] The name of the weight column in your Cooler URI for normalizing the contact signals.: "The name of the weight column in your Cooler URI for normalizing the contact
                            signals."
