---
name: false-positive-annotation-against-chromatin-structure
description: Use when you have genome-wide in vitro TF binding predictions and a set
  of experimental in vivo peaks, and need to quantify TP rate, filter corroborated
  false positives, and test whether remaining FPs sit in nucleosome-occupied regions.
when_to_use_negative:
- You lack experimental in vivo peak data (ChIP-exo/PB-exo) to define true positives,
  so TP/FP rates cannot be computed.
- Your target organism has no nucleosome positioning maps available — chromatin-structure
  explanation of false positives cannot be performed.
- The transcription factor has no gcPBM training data compatible with gcpbm_regressor.py's
  required file format.
tools:
- name: DNAffinity
  role: gcPBM-trained random forest regressor (gcpbm_regressor.py) used to score genome-wide
    CBF1 binding affinity before thresholding into a TFBS map
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:0728a53bd4fda07684a972ef86a18a014f306063f79dcb5f4d9d5f22289119e2
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# false-positive-annotation-against-chromatin-structure

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation workflow that judges an in vitro-trained transcription factor binding site predictor (DNAffinity gcpbm_regressor) against in vivo peaks (ChIP-exo GSE44604, PB-exo GSE147927) in S. cerevisiae, computing true-positive rate and then explaining residual false positives via G1-phase nucleosome occupancy maps.

## When to use

Use when you have genome-wide in vitro TF binding predictions and a set of experimental in vivo peaks, and need to quantify TP rate, filter corroborated false positives, and test whether remaining FPs sit in nucleosome-occupied regions.

## When NOT to use

- You lack experimental in vivo peak data (ChIP-exo/PB-exo) to define true positives, so TP/FP rates cannot be computed.
- Your target organism has no nucleosome positioning maps available — chromatin-structure explanation of false positives cannot be performed.
- The transcription factor has no gcPBM training data compatible with gcpbm_regressor.py's required file format.

## Inputs

- DNAffinity gcPBM regressor input files: proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt (template format)
- ChIP-exo peak coordinates (GSE44604) for CBF1 in S. cerevisiae
- PB-exo peak coordinates (GSE147927) for CBF1 in S. cerevisiae
- Yeast G1-phase nucleosome occupancy maps
- Genome-wide predicted TFBS coordinates from thresholded DNAffinity affinity scores

## Outputs

- True-positive rate of predicted sites against exo peaks (reported value >94% TP/total exo peaks)
- Real false-positive rate after corroborating-evidence filtering (reported ~7%)
- Fraction of retained false positives coinciding with nucleosome-occupied regions

## How to apply

First score binding affinity genome-wide by running the DNAffinity gcPBM regressor: supply proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt in the template format, then run gcpbm_regressor.py with the protein name as 1st argument and the concentration as 2nd. Threshold the affinity scores into a discrete TFBS map across the S. cerevisiae genome. Intersect predicted sites with ChIP-exo (GSE44604) and PB-exo (GSE147927) peak coordinates to label true positives and compute the TP rate over total exo peak counts. Filter remaining predictions against other experimental evidence to obtain the real false-positive rate, then overlay retained false positives with yeast G1-phase nucleosome occupancy maps to quantify the fraction located in nucleosome-occupied regions.

## Related tools

- **DNAffinity** (gcPBM-trained random forest regressor (gcpbm_regressor.py) used to score genome-wide CBF1 binding affinity before thresholding into a TFBS map) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python gcpbm_regressor.py CBF1 {concentration}
```

## Evaluation signals

- Reproduced true-positive rate matches the reported value (TP/total number of exo experiment peaks over 94%, i.e. fewer than 6% of consensus exo peaks missed).
- Real false-positive rate after filtering predictions corroborated by other experimental evidence is approximately 7%.
- Most retained false positives coincide with nucleosome-occupied locations in the G1-phase yeast nucleosome maps.
- Input files match the template formatting required by gcpbm_regressor.py (compare with templates per the README).

## Limitations

- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists, so apparent false positives may not be true errors.
- The concept of a transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy and low-resolution experimental technique used to detect it.
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity.

## Evidence
