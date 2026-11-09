---
name: nucleosome-occupancy-filtering-of-predictions
description: Explains and contextualizes predicted TF binding sites that lack experimental
  support by overlaying them with G1-phase yeast nucleosome occupancy maps, as done
  for CBF1 false positives from DNAffinity PB-exo/ChIP-exo predictions.
when_to_use_negative:
- When the goal is in vitro affinity prediction accuracy against uPBM/gcPBM/HT-SELEX
  data (R2 evaluation), which requires no nucleosome or in vivo peak data
- When nucleosome occupancy maps for the organism/cell phase are unavailable or the
  TF's in vivo context is not yeast chromatin
- When predictions already coincide with experimental peaks (true positives), where
  chromatin filtering adds no information
tools:
- name: DNAffinity
  role: Generates the gcPBM-trained genome-wide CBF1 affinity predictions (gcpbm_regressor.py)
    that are thresholded and intersected with exo peaks before nucleome-occupancy
    annotation of false positives
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:4fce36748fd65e55d69182316add974f8f5ace3f2bdb2b46d5887b7f45ba6976
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
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nucleosome-occupancy-filtering-of-predictions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Explains and contextualizes predicted TF binding sites that lack experimental support by overlaying them with G1-phase yeast nucleosome occupancy maps, as done for CBF1 false positives from DNAffinity PB-exo/ChIP-exo predictions.

## When to use

Use when an in vitro-trained predictor (e.g. DNAffinity gcpbm_regressor on CBF1) yields genome-wide predicted sites, many of which do not overlap ChIP-exo (GSE44604) or PB-exo (GSE147927) peaks, and you must quantify how many 'false positives' fall in nucleosome-occupied chromatin.

## When NOT to use

- When the goal is in vitro affinity prediction accuracy against uPBM/gcPBM/HT-SELEX data (R2 evaluation), which requires no nucleosome or in vivo peak data
- When nucleosome occupancy maps for the organism/cell phase are unavailable or the TF's in vivo context is not yeast chromatin
- When predictions already coincide with experimental peaks (true positives), where chromatin filtering adds no information

## Inputs

- Genome-wide DNAffinity affinity scores (gcpbm_regressor.py output for CBF1, from proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt)
- ChIP-exo peak coordinates for CBF1 (GSE44604)
- PB-exo peak coordinates for CBF1 (GSE147927)
- Yeast G1-phase nucleosome occupancy maps (references 5,54)

## Outputs

- True-positive rate of predicted consensus exo peaks (reported >94%)
- Real false-positive rate after corroborating-evidence filtering (~7%)
- Fraction of retained false positives coinciding with nucleosome-occupied regions
- Discrete TFBS map thresholded from genome-wide affinity scores

## How to apply

First generate the in vitro affinity map: load the gcPBM-trained CBF1 regressor from DNAffinity (gcpbm_regressor.py) with proteins/CBF1/CBF1_{concentration}.txt and proteins/CBF1/_freq_matrix_6.txt, then threshold genome-wide scores into a discrete TFBS map. Intersect predicted sites with ChIP-exo (GSE44604) and PB-exo (GSE147907) peak coordinates to label true positives and compute the true-positive rate (reported >94%). For the retained predictions lacking exo support, filter those corroborated by other evidence to estimate the real false-positive rate (~7%). Finally, overlay the remaining false positives with yeast G1-phase nucleosome occupancy maps to quantify the fraction located in nucleosome-occupied regions; the rationale is that chromatin structure, not predictor error, explains most non-overlapping predictions.

## Related tools

- **DNAffinity** (Generates the gcPBM-trained genome-wide CBF1 affinity predictions (gcpbm_regressor.py) that are thresholded and intersected with exo peaks before nucleome-occupancy annotation of false positives) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python gcpbm_regressor.py CBF1 {concentration}
```

## Evaluation signals

- Reproduced true-positive rate matches the reported >94% (fewer than 6% of consensus exo peaks missed)
- Real false-positive rate after corroborating-evidence filtering is ~7%
- Majority of retained false positives coincide with nucleosome-occupied locations in G1-phase maps
- Intersections use the correct accessions: ChIP-exo GSE44604 and PB-exo GSE147927
- Input files match the DNAffinity template formats (proteins/CBF1/CBF1_{concentration}.txt and _freq_matrix_6.txt)

## Limitations

- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic TF affinity
- The concept of transcription factor binding site is not uniquely defined, depending on the noisy, low-resolution experimental technique used to detect it, so a universal predictor is impossible

## Evidence
