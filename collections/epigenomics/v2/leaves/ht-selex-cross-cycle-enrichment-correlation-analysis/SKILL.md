---
name: ht-selex-cross-cycle-enrichment-correlation-analysis
description: Use when running selex_regressor.py (DNAffinity) across multiple HT-SELEX
  cycles and you need to detect/remove cases whose enrichment between cycles is inconsistent
  before training or evaluating the regressor.
when_to_use_negative:
- Data is uPBM or gcPBM — inconsistent-cycle filtering is specific to multi-cycle
  HT-SELEX; uPBM instead requires sequence trimming based on the MEME suite result
- Only a single HT-SELEX cycle is available — no cross-cycle correlation descriptors
  can be computed for the SVM discriminant
- Cases are already confirmed consistent or you have a curated high-quality training
  set — filtering would remove data without benefit
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DNAffinity
  role: Runs the selex_regressor.py random forest pipeline on per-cycle HT-SELEX count
    files; the SVM filtering is layered on top of its training/evaluation loop to
    remove inconsistent cases
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:c9392726e82a8c32eb5f1574dcc5ba3ef15a96b062c5fc7eccd64a30b1d38679
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ht-selex-cross-cycle-enrichment-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-filtering procedure for HT-SELEX-based affinity prediction: correlations of selected sequence counts across SELEX cycles train an SVM discriminant that flags inconsistent cases, which are removed before training the DNAffinity random forest regressor. This raised average R2 on HT-SELEX data from 0.66 ± 0.19 to 0.70 ± 0.14.

## When to use

Use when running selex_regressor.py (DNAffinity) across multiple HT-SELEX cycles and you need to detect/remove cases whose enrichment between cycles is inconsistent before training or evaluating the regressor.

## When NOT to use

- Data is uPBM or gcPBM — inconsistent-cycle filtering is specific to multi-cycle HT-SELEX; uPBM instead requires sequence trimming based on the MEME suite result
- Only a single HT-SELEX cycle is available — no cross-cycle correlation descriptors can be computed for the SVM discriminant
- Cases are already confirmed consistent or you have a curated high-quality training set — filtering would remove data without benefit

## Inputs

- Per-cycle HT-SELEX count files: proteins/{protein}/{cycle}.txt (formatting checked against repository templates)
- Selected sequence counts across multiple SELEX cycles (e.g. 5-cycle and 7-cycle HT-SELEX datasets)
- DNAffinity package with selex_regressor.py and its feature parameter database

## Outputs

- SVM discriminant flags marking HT-SELEX cases with inconsistent cross-cycle enrichments
- Filtered HT-SELEX training set (suspicious cases removed) used to retrain the random forest regressor
- Per-case R2 on the held-out 20% of sequences and the average R2 after filtering (paper benchmark: 0.70 ± 0.14)
- Affinity predictions written to selex_affinity_predictions.csv

## How to apply

Prepare per-cycle HT-SELEX count files at proteins/{protein}/{cycle}.txt, verifying formatting against the repository templates. For each protein, launch selex_regressor.py with the protein name as the 1st argument and the cycle number as the 2nd (the bundled test case validates the install). Compute correlations of selected sequence counts across SELEX cycles and use them as descriptors to train an SVM discriminant; cases flagged as displaying inconsistency between enrichments at different cycles are treated as suspicious and removed, since they would bias training and testing. Retrain the random forest regressor on the filtered HT-SELEX set, then compute R2 between predictions and experimental affinities on the held-out 20% of sequences and compare against the unfiltered baseline (0.66 ± 0.19 average R2; 0.63 ± 0.19 for 5-cycle and 0.71 ± 0.21 for 7-cycle data), writing predictions to selex_affinity_predictions.csv.

## Related tools

- **DNAffinity** (Runs the selex_regressor.py random forest pipeline on per-cycle HT-SELEX count files; the SVM filtering is layered on top of its training/evaluation loop to remove inconsistent cases) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Average R2 after SVM filtering should approach the paper benchmark of 0.70 ± 0.14, versus the unfiltered baseline of 0.66 ± 0.19 (0.63 ± 0.19 for 5-cycle and 0.71 ± 0.21 for 7-cycle data)
- A dramatic reduction in the number of cases with low R2 after removing SVM-flagged suspicious cases
- R2 is computed only on the held-out 20% of sequences (model trained on 80%), as in the article protocol
- Input count files match the required proteins/{protein}/{cycle}.txt formatting compared against repository templates
- Predictions file selex_affinity_predictions.csv is produced by the selex_regressor.py run

## Limitations

- Cases flagged by the SVM are discarded as suspicious rather than repaired, so over-aggressive filtering could remove recoverable data
- The reported unfiltered HT-SELEX performance is noisy (0.66 ± 0.19), so improvements must be assessed on the distribution of R2, not the mean alone
- File-format deviations (e.g. count files not matching the proteins/{protein}/{cycle}.txt template) will break the regressor run before filtering can be applied

## Evidence
