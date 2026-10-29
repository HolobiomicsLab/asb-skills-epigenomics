---
name: genome-wide-tfbs-prediction-scanning
description: Use when an in vitro trained TF affinity model (e.g. DNAffinity on gcPBM
  data for CBF1) must be applied genome-wide to predict in vivo binding sites and
  validated against exo peak calls.
when_to_use_negative:
- The goal is to predict in vitro binding affinities from uPBM or HT-SELEX data —
  use upbm_regressor.py or selex_regressor.py pipelines instead of the gcPBM genome-wide
  scan
- No exo (ChIP-exo/PB-exo) peak data or nucleosome maps exist for the organism, so
  TP/FP validation and chromatin-structure correction cannot be performed
- The transcription factor of interest is not represented in the DNAffinity protein
  parameter database (no gcPBM training data and frequency matrix available)
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DNAffinity
  role: gcPBM-trained random forest regressor (gcpbm_regressor.py) that scores TF
    binding affinity genome-wide from DNA physical properties
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:c8adfdc6083231dd969d28d6b762fb704ae28e0ff0401f8d75ba6a9a9e5df0da
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genome-wide-tfbs-prediction-scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores transcription factor binding affinity genome-wide using the DNAffinity gcPBM-trained random forest regressor, thresholds the scores into a discrete TFBS map, and validates it against ChIP-exo (GSE44604) and PB-exo (GSE147927) peaks with nucleosome occupancy overlays. Demonstrated for CBF1 in S. cerevisiae, achieving >94% true positive rate and a ~7% real false positive rate.

## When to use

Use when an in vitro trained TF affinity model (e.g. DNAffinity on gcPBM data for CBF1) must be applied genome-wide to predict in vivo binding sites and validated against exo peak calls.

## When NOT to use

- The goal is to predict in vitro binding affinities from uPBM or HT-SELEX data — use upbm_regressor.py or selex_regressor.py pipelines instead of the gcPBM genome-wide scan
- No exo (ChIP-exo/PB-exo) peak data or nucleosome maps exist for the organism, so TP/FP validation and chromatin-structure correction cannot be performed
- The transcription factor of interest is not represented in the DNAffinity protein parameter database (no gcPBM training data and frequency matrix available)

## Inputs

- proteins/{protein}/{protein}_{concentration}.txt training file (gcPBM format, template-checked)
- proteins/{protein}/_freq_matrix_6.txt frequency matrix
- Protein name and concentration as command-line arguments (e.g. CBF1)
- ChIP-exo peak coordinates (GSE44604) and PB-exo peak coordinates (GSE147927) for S. cerevisiae
- Yeast G1-phase nucleosome occupancy maps for chromatin-structure filtering

## Outputs

- Genome-wide binding affinity scores for the transcription factor
- Discrete TFBS map across the S. cerevisiae genome (thresholded affinity scores)
- True-positive rate (TP / total number of exo experiment peaks; CBF1 reported >94%)
- Real false-positive rate after corroborating-evidence filtering (CBF1 reported ~7%)
- Fraction of false positives located in nucleosome-occupied regions

## How to apply

Load the gcPBM-trained regressor from the DNAffinity package (gcpbm_regressor.py), supplying proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt in the template format; run it passing the protein name as the 1st argument and the concentration as the 2nd to score binding affinity genome-wide. Threshold the genome-wide affinity scores into a discrete TFBS map across the S. cerevisiae genome. Intersect predicted sites with ChIP-exo (GSE44604) and PB-exo (GSE147927) peak coordinates to label true positives and compute the true-positive rate (TP/total number of exo experiment peaks, reported >94% for CBF1). Filter the remaining predictions against other experimental evidence to obtain the real false-positive rate (~7% for CBF1), then overlay retained false positives with G1-phase yeast nucleosome occupancy maps to quantify the fraction in nucleosome-occupied regions. The rationale is that in vitro affinity alone overpredicts, so chromatin structure information (nucleosome positioning) complements the prediction for in vivo TFBS identification.

## Related tools

- **DNAffinity** (gcPBM-trained random forest regressor (gcpbm_regressor.py) that scores TF binding affinity genome-wide from DNA physical properties) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python gcpbm_regressor.py CBF1 <concentration>
```

## Evaluation signals

- Reproduced true-positive rate matches the reported value: TP / total number of exo experiment peaks > 94% for CBF1 (fewer than 6% of consensus exo peaks missed)
- Real false-positive rate after corroborating-evidence filtering is ~7%, matching the reported value
- Most retained false positives coincide with nucleosome-occupied locations in the G1-phase yeast nucleosome maps
- Input files pass the template-format check: proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt present with correct formatting
- gcPBM training quality for the protein matches reported performance (e.g. CBF1 R2 = 0.80; gcPBM average R2 = 0.93 ± 0.02 across TFs)

## Limitations

- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists; in vitro affinity alone overpredicts in vivo sites
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity (chromatin, cofactors)
- The concept of transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy and low-resolution experimental technique used to detect it, preventing a universal predictor

## Evidence
