---
name: chip-exo-peak-overlap-analysis
description: Use when you have genome-wide binding-affinity predictions from an in
  vitro-trained regressor (e.g. DNAffinity gcPBM model for CBF1) and experimental
  peak coordinates (ChIP-exo/PB-exo BED-like intervals) to compute TP and FP rates
  in vivo.
when_to_use_negative:
- You only have in vitro affinity data (uPBM, gcPBM, HT-SELEX) with no in vivo peak
  sets to intersect — the in vitro R2 evaluation workflow applies instead
- Your target organism lacks nucleosome positioning maps, so false positives cannot
  be explained by chromatin structure
- Your TF has no gcPBM training files in the required template format for gcpbm_regressor.py
tools:
- name: DNAffinity
  role: gcPBM-trained random forest regressor (gcpbm_regressor.py) scoring genome-wide
    binding affinity for CBF1 before thresholding into a TFBS map
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:8dec0fe0b475dafd02ee012368b2bd2a45e70a69bdf9c8e66ec31d5549c2bb66
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Jalbiti/DNAffinity
  repo_url: https://github.com/Jalbiti/DNAffinity
  tools:
  - DNAffinity
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# chip-exo-peak-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates an in vitro-trained DNAffinity binding-affinity predictor by thresholding genome-wide scores into a TFBS map and intersecting predicted sites with ChIP-exo (GSE44604) and PB-exo (GSE147927) peaks along the S. cerevisiae genome, then explaining false positives with G1-phase nucleosome occupancy.

## When to use

Use when you have genome-wide binding-affinity predictions from an in vitro-trained regressor (e.g. DNAffinity gcPBM model for CBF1) and experimental peak coordinates (ChIP-exo/PB-exo BED-like intervals) to compute TP and FP rates in vivo.

## When NOT to use

- You only have in vitro affinity data (uPBM, gcPBM, HT-SELEX) with no in vivo peak sets to intersect — the in vitro R2 evaluation workflow applies instead
- Your target organism lacks nucleosome positioning maps, so false positives cannot be explained by chromatin structure
- Your TF has no gcPBM training files in the required template format for gcpbm_regressor.py

## Inputs

- gcPBM-trained DNAffinity regressor input files: proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt
- ChIP-exo peak coordinates for CBF1 in yeast (GEO GSE44604)
- PB-exo peak coordinates for CBF1 in yeast (GEO GSE147927)
- Yeast G1-phase nucleosome occupancy maps
- S. cerevisiae genome sequence / coordinates for genome-wide affinity scoring

## Outputs

- Discrete TFBS map (predicted binding sites) across the S. cerevisiae genome
- True-positive rate (TP / total number of exo experiment peaks)
- Real false-positive rate after corroborating-evidence filtering (~7% reported)
- Fraction of false positives overlapping nucleosome-occupied regions

## How to apply

Run the DNAffinity gcpbm_regressor.py with the protein name (CBF1) as the 1st argument and the concentration as the 2nd argument, having supplied proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt, to score binding affinity genome-wide. Threshold the resulting affinity scores into a discrete TFBS map across the S. cerevisiae genome. Intersect predicted sites with ChIP-exo (GSE44604) and PB-exo (GSE147927) peak coordinates to label true positives and compute the true-positive rate (TP/total number of exo experiment peaks). For the remaining predictions, filter those corroborated by other experimental evidence and compute the real false-positive rate, then overlay retained false positives with yeast G1-phase nucleosome occupancy maps to quantify the fraction located in nucleosome-occupied regions.

## Related tools

- **DNAffinity** (gcPBM-trained random forest regressor (gcpbm_regressor.py) scoring genome-wide binding affinity for CBF1 before thresholding into a TFBS map) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python gcpbm_regressor.py CBF1 <concentration>
```

## Evaluation signals

- Reproduced true-positive rate matches the reported value (over 94% of consensus exo peaks recovered)
- Real false-positive rate after corroborating-evidence filtering is approximately the reported ~7%
- Most retained false positives coincide with nucleosome-occupied locations from G1-phase yeast maps
- Predicted-site interval format is consistent with the ChIP-exo/PB-exo peak coordinates so intersection yields non-empty overlap sets

## Limitations

- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity
- The concept of transcription factor binding site is not uniquely defined, as it depends on the intrinsically noisy and low-resolution experimental technique used to detect it

## Evidence

- [results] After training our regressor using gcPBM data (R2 = 0.80), we applied our model to predict PB-exo and ChIP-exo peaks along the yeast genome: "After training our regressor using gcPBM data (R2 = 0.80), we applied our model to predict PB-exo and ChIP-exo peaks along the yeast genome"
- [results] To account for the impact of chromatin structure, we include accurate nucleosome maps collected for yeast in the G1 phase (5,54): "To account for the impact of chromatin structure, we include accurate nucleosome maps collected for yeast in the G1 phase (5,54)"
- [discussion] When combining DNAffinity and nucleosome maps, our method was able to locate in vivo TFBS with a high accuracy.: "When combining DNAffinity and nucleosome maps, our method was able to locate in vivo TFBS with a high accuracy."
- [readme] proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt for `gcpbm_regressor.py`: "proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt for `gcpbm_regressor.py`"
