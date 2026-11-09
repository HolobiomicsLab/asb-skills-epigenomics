---
name: ht-selex-affinity-regression
description: Use when HT-SELEX per-cycle enrichment files (proteins/{protein}/{cycle}.txt,
  e.g. from ENA PRJEB14744 or PRJEB29730) are available and per-TF k-mer binding affinities
  must be predicted or benchmarked.
when_to_use_negative:
- The task targets uPBM or gcPBM data — use upbm_regressor.py or gcpbm_regressor.py
  with their own file formats (e.g. proteins/{protein}/{protein}_training.txt) instead
- A transcription factor case fails the SVM consistency check between enrichments
  at different HT-SELEX cycles — it must be discarded rather than trained or tested
  on
- No per-cycle HT-SELEX enrichment files exist and only cross-platform transfer to
  uPBM is needed (that requires the additional upbm_regressor.py workflow with MEME-based
  sequence trimming)
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DNAffinity
  role: Provides selex_regressor.py, the random forest regressor trained on MD-derived
    DNA physical-property features to predict HT-SELEX affinities
  repo: https://github.com/Jalbiti/DNAffinity
- name: scikit-learn
  role: Implements the random forest regressor and the SVM-based filtering of inconsistent
    HT-SELEX cycle enrichments
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:99f61d251d152fc7700f7dc00cc1c914af07b3fa6d64d3719680a56fb0de2705
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
  - BEESEM
  - BindSpace
  - DNAffinity
  - DeepBind
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ht-selex-affinity-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the DNAffinity random-forest regressor for HT-SELEX data: DNA physical-property features (equilibrium geometry and flexibility at base-pair-step resolution from MD-derived parameters) are used to predict per-protein k-mer binding affinities and validate against published R2 benchmarks (0.63 ± 0.19 on 5-cycle, 0.71 ± 0.21 on 7-cycle, 0.66 ± 0.19 combined).

## When to use

Use when HT-SELEX per-cycle enrichment files (proteins/{protein}/{cycle}.txt, e.g. from ENA PRJEB14744 or PRJEB29730) are available and per-TF k-mer binding affinities must be predicted or benchmarked.

## When NOT to use

- The task targets uPBM or gcPBM data — use upbm_regressor.py or gcpbm_regressor.py with their own file formats (e.g. proteins/{protein}/{protein}_training.txt) instead
- A transcription factor case fails the SVM consistency check between enrichments at different HT-SELEX cycles — it must be discarded rather than trained or tested on
- No per-cycle HT-SELEX enrichment files exist and only cross-platform transfer to uPBM is needed (that requires the additional upbm_regressor.py workflow with MEME-based sequence trimming)

## Inputs

- HT-SELEX per-cycle enrichment files formatted as proteins/{protein}/{cycle}.txt (ENA accessions PRJEB14744 and PRJEB29730)
- Penultimate-cycle k-mer sequences and their experimental affinity values per protein
- Protein name and SELEX cycle number passed as command-line arguments

## Outputs

- Per-protein affinity predictions on the held-out 20% of k-mers
- Per-protein R2, Pearson correlation and MSE metrics aggregated into selex_r2_metrics.csv
- Average R2 for the 5-cycle dataset, the 7-cycle dataset, and the combined total

## How to apply

Pre-filter the HT-SELEX datasets with an SVM-based check, removing transcription factor cases that show inconsistency between enrichments at different SELEX cycles so suspicious cases do not bias training or testing. For each protein, extract DNA physical-property features (equilibrium geometry and flexibility at base-pair-step resolution from the MD-derived parameter database) for k-mers in the penultimate-cycle sequences. Train the random forest regressor with selex_regressor.py, passing the protein name as the 1st argument and the cycle number as the 2nd, on 80% of the k-mer feature vectors against penultimate-cycle affinity values. Evaluate on the held-out 20% by computing R2, Pearson correlation and MSE per protein, then aggregate per-protein R2 values into averages for the 5-cycle dataset, the 7-cycle dataset, and the combined total (write metrics to selex_r2_metrics.csv). Agreement with the published values (0.63 ± 0.19 for 5-cycle, 0.66 ± 0.19 combined) within the reported standard deviation is the acceptance criterion.

## Related tools

- **DNAffinity** (Provides selex_regressor.py, the random forest regressor trained on MD-derived DNA physical-property features to predict HT-SELEX affinities) — https://github.com/Jalbiti/DNAffinity
- **scikit-learn** (Implements the random forest regressor and the SVM-based filtering of inconsistent HT-SELEX cycle enrichments)

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Average R2 on the 5-cycle dataset reproduces 0.63 ± 0.19 and on the 7-cycle dataset 0.71 ± 0.21, with a combined total average of 0.66 ± 0.19 within the reported standard deviation
- The HT-SELEX-to-uPBM transfer benchmark yields an average Pearson correlation of ~0.47, outperforming DeepSELEX (0.41), BEESEM (0.36), DeepBind (0.35) and BindSpace (0.20)
- Training used 80% of k-mer feature vectors and metrics were computed on the held-out 20% per protein
- SVM filtering removed TF cases with inconsistent enrichments between SELEX cycles before training/testing
- All per-protein R2, Pearson and MSE values are written to selex_r2_metrics.csv

## Limitations

- TF cases displaying inconsistency between enrichments at different HT-SELEX cycles are suspicious and must be removed via SVM filtering, otherwise they bias training and testing
- The transcription factor binding site concept is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique used to detect it, preventing a universal predictor
- The regressor uses the penultimate-cycle sequences; input files must match the proteins/{protein}/{cycle}.txt template formatting exactly

## Evidence
