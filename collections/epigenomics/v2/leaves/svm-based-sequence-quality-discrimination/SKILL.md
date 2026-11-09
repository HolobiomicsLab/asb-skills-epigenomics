---
name: svm-based-sequence-quality-discrimination
description: Use when reproducing DNAffinity on HT-SELEX data (per-cycle proteins/{protein}/{cycle}.txt
  files) and cases show inconsistency between enrichments at different SELEX cycles,
  biasing training/testing.
when_to_use_negative:
- Data is uPBM or gcPBM, where no multi-cycle enrichment consistency check applies
  (uPBM sequences are instead trimmed based on MEME suite results)
- Only a single HT-SELEX cycle is available, so cross-cycle enrichment correlations
  cannot be computed as SVM descriptors
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Package providing selex_regressor.py, run with protein name and cycle number,
    whose multi-cycle HT-SELEX enrichments feed the SVM quality filter and downstream
    random forest regressor
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:dad59ce4f97f2dd66023fbbffc7a221d359f1f8609a515ddaa4d791b8ed18878
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# svm-based-sequence-quality-discrimination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An SVM-based filtering skill from DNAffinity that flags suspicious HT-SELEX cases by training a discriminant on correlations of selected sequence counts across SELEX cycles, so that inconsistent enrichments between cycles can be removed before training/testing the random forest affinity regressor.

## When to use

Use when reproducing DNAffinity on HT-SELEX data (per-cycle proteins/{protein}/{cycle}.txt files) and cases show inconsistency between enrichments at different SELEX cycles, biasing training/testing.

## When NOT to use

- Data is uPBM or gcPBM, where no multi-cycle enrichment consistency check applies (uPBM sequences are instead trimmed based on MEME suite results)
- Only a single HT-SELEX cycle is available, so cross-cycle enrichment correlations cannot be computed as SVM descriptors

## Inputs

- HT-SELEX per-cycle count files proteins/{protein}/{cycle}.txt
- Correlations of selected sequence counts across SELEX cycles (SVM descriptors)

## Outputs

- SVM-flagged list of suspicious/inconsistent HT-SELEX cases
- Filtered HT-SELEX training set
- Affinity predictions written to selex_affinity_predictions.csv
- R2 on the held-out 20% of sequences

## How to apply

First prepare per-cycle HT-SELEX count files in the DNAffinity repository format (proteins/{protein}/{cycle}.txt) and verify formatting against the templates. Launch selex_regressor.py with the protein name as the 1st argument and cycle number as the 2nd argument (or use the bundled test case). Compute correlations of selected sequence counts across SELEX cycles and use these correlations as descriptors to train an SVM discriminant that flags cases with inconsistent enrichments between cycles, since such cases are suspicious. Remove the SVM-flagged cases and retrain the random forest regressor on the filtered HT-SELEX training set, evaluating R2 on the held-out 20% of sequences. In the paper this raised average R2 from an unfiltered 0.66 ± 0.19 to 0.70 ± 0.14, with a dramatic reduction of low-R2 cases; write predictions to selex_affinity_predictions.csv.

## Related tools

- **DNAffinity** (Package providing selex_regressor.py, run with protein name and cycle number, whose multi-cycle HT-SELEX enrichments feed the SVM quality filter and downstream random forest regressor) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Average R2 after SVM filtering should improve over the unfiltered HT-SELEX baseline (paper: 0.66 ± 0.19 → 0.70 ± 0.14)
- Standard deviation of R2 should shrink after filtering (0.19 → 0.14), with a dramatic reduction of cases with low R2
- SVM descriptors are correlations of selected counts across SELEX cycles; flagged cases must show inconsistency between enrichments at different HT-SELEX cycles
- R2 computed between predictions and experimental affinities on the held-out 20% of sequences (80/20 train/test split)

## Limitations

- Filtering is specific to multi-cycle HT-SELEX; unfiltered baselines differ by cycle count (0.63 ± 0.19 for 5-cycle and 0.71 ± 0.21 for 7-cycle data)
- SVM-flagged cases are 'suspicious' but the filter discards data rather than correcting it, reducing usable cases
- The transcription factor binding site concept is technique-dependent and noisy, so filtered HT-SELEX-trained models may not transfer across experimental techniques

## Evidence

- [results] Unfiltered HT-SELEX baseline and filtered R2 improvement: [evidence span withheld — anchor status: not_found]
