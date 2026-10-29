---
name: selex-cycle-enrichment-consistency-filtering
description: Use when preparing HT-SELEX datasets (e.g. ENA PRJEB14744 or PRJEB29730)
  for affinity regression and you need to exclude TFs whose enrichment values are
  inconsistent across SELEX cycles before training or testing.
when_to_use_negative:
- Input data is uPBM or gcPBM — this SVM cycle-consistency filter is specific to multi-cycle
  HT-SELEX enrichments and does not apply
- You have only a single SELEX cycle of enrichment data, so no cross-cycle consistency
  comparison is possible
- The TF files do not yet match the template formatting expected by the DNAffinity
  regressor (proteins/{protein}/{cycle}.txt), since the filter assumes correctly formatted
  inputs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Pipeline whose HT-SELEX branch includes the SVM consistency filter before
    random forest affinity regression; run via selex_regressor.py with protein name
    and cycle number arguments
  repo: https://github.com/Jalbiti/DNAffinity
- name: scikit-learn
  role: Provides the SVM used to detect inconsistency between enrichments at different
    HT-SELEX cycles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:8df16aa91e3b0ada5082d4ef6070000765749b4969e1c2568b10e49802010282
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# selex-cycle-enrichment-consistency-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An SVM-based pre-filtering step that removes transcription factor (TF) cases showing inconsistency between enrichments at different HT-SELEX cycles, so that suspicious cases do not bias training or evaluation of the DNAffinity random forest regressor.

## When to use

Use when preparing HT-SELEX datasets (e.g. ENA PRJEB14744 or PRJEB29730) for affinity regression and you need to exclude TFs whose enrichment values are inconsistent across SELEX cycles before training or testing.

## When NOT to use

- Input data is uPBM or gcPBM — this SVM cycle-consistency filter is specific to multi-cycle HT-SELEX enrichments and does not apply
- You have only a single SELEX cycle of enrichment data, so no cross-cycle consistency comparison is possible
- The TF files do not yet match the template formatting expected by the DNAffinity regressor (proteins/{protein}/{cycle}.txt), since the filter assumes correctly formatted inputs

## Inputs

- HT-SELEX per-protein per-cycle sequence/enrichment files in the proteins/{protein}/{cycle}.txt format required by selex_regressor.py
- Enrichment values measured at multiple SELEX cycles for each TF (5-cycle and 7-cycle experiments)
- ENA-derived datasets from accessions PRJEB14744 and PRJEB29730

## Outputs

- A filtered set of TF cases (consistent across SELEX cycles) suitable for random forest regressor training and testing
- Downstream per-protein metrics (R2, Pearson correlation, MSE) written to selex_r2_metrics.csv for the filtered 5-cycle and 7-cycle datasets

## How to apply

Collect the HT-SELEX training files for each protein from ENA accessions PRJEB14744 and PRJEB29730, verifying each proteins/{protein}/{cycle}.txt file matches the template formatting required by the regressor. Before feature extraction and random forest training, run an SVM-based consistency check across the enrichment values measured at different SELEX cycles for each TF. Remove TF cases flagged by the SVM as displaying inconsistency between cycle enrichments, since the article treats these as 'suspicious cases' that would bias both the 80% training split and the 20% held-out test split. Only after filtering, extract DNA physical-property features (equilibrium geometry and flexibility at base-pair-step resolution from the MD-derived parameter database) for the k-mers of the penultimate-cycle sequences and train selex_regressor.py with the protein name and cycle number as arguments.

## Related tools

- **DNAffinity** (Pipeline whose HT-SELEX branch includes the SVM consistency filter before random forest affinity regression; run via selex_regressor.py with protein name and cycle number arguments) — https://github.com/Jalbiti/DNAffinity
- **scikit-learn** (Provides the SVM used to detect inconsistency between enrichments at different HT-SELEX cycles)

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- After filtering, the average R2 on the HT-SELEX regressor should match the published reference values: 0.63 ± 0.19 on the 5-cycle dataset, 0.71 ± 0.21 on the 7-cycle dataset, and 0.66 ± 0.19 combined — agreement within the reported standard deviation is the acceptance criterion
- Every TF case flagged as inconsistent across cycles by the SVM should be excluded from both the 80% training split and the 20% test split
- Input files must verify against the template formatting: proteins/{protein}/{cycle}.txt for selex_regressor.py
- All computed metrics (R2, Pearson, MSE per protein, plus 5-cycle/7-cycle/combined averages) should be present in selex_r2_metrics.csv

## Limitations

- The filter is heuristic: the article calls inconsistent cross-cycle cases 'suspicious cases', so legitimate TFs may be removed along with genuinely noisy ones
- Transcription factor binding site data is intrinsically noisy and technique-dependent ('the concept of transcription factor binding site is not uniquely defined, as it deeply depends on the intrinsically noisy and low-resolution experimental technique used to detect it'), which limits what filtering can resolve
- Filtering requires multiple SELEX cycles of enrichment data per TF; single-cycle data cannot be assessed for cross-cycle consistency

## Evidence

- [results] The HT-SELEX regressor achieved an R2 of 0.63 ± 0.19 on the 5-cycle dataset, 0.71 ± 0.21 on the 7-cycle dataset, and a combined total average of 0.66 ± 0.19.: [evidence span withheld — anchor status: not_found]
