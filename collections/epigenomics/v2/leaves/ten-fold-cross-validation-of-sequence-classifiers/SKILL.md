---
name: ten-fold-cross-validation-of-sequence-classifiers
description: Use when reproducing or extending promoter-prediction benchmarking that
  requires ten-fold cross-validation of ensemble sequence classifiers (RF + CNN +
  weighted average) on labeled promoter/non-promoter FASTA-style training sets.
when_to_use_negative:
- Input is already an extracted feature matrix rather than raw promoter/non-promoter
  sequences — skip feature_code.py/iLearnPlus steps
- Evaluating on independent test sets or cross-cell-line prediction rather than training-set
  cross-validation
- Working with non-human species where the human word2vec benchmark files (index_promoters.txt,
  word2vec_promoters.txt) do not apply
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_3474
tools:
- name: iPro-WAEL
  role: weighted average ensemble learning framework orchestrating the cross-validation
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: keras 2.3.1
  role: backend for the CNN base model (CNN.py)
- name: tensorflow 2.0.0
  role: backend for the CNN base model (CNN.py)
- name: sklearn
  role: random forest base model (RF.py) and cross-validation splitting
- name: iLearnPlus
  role: extraction of additional sequence features beyond word2vec
- name: Python 3.6
  role: runtime for feature_code.py, RF.py, CNN.py, Weighted_average.py and main.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:71a7d7fa891f7c9dd06c315e9346f16572d5064fe7ed2c7c675c5b7482122d2c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  repo_url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  tools:
  - Python
  - Python 3.6
  - h5py
  - iLearnPlus
  - iPro-WAEL
  - keras
  - keras 2.3.1
  - numpy
  - sklearn
  - tensorflow
  - tensorflow 2.0.0
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# ten-fold-cross-validation-of-sequence-classifiers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the iPro-WAEL ten-fold cross-validation protocol, in which a random forest and a CNN base model are combined by a weighted average algorithm to identify promoters, evaluating sensitivity, specificity, accuracy and MCC per cell line or species.

## When to use

Use when reproducing or extending promoter-prediction benchmarking that requires ten-fold cross-validation of ensemble sequence classifiers (RF + CNN + weighted average) on labeled promoter/non-promoter FASTA-style training sets.

## When NOT to use

- Input is already an extracted feature matrix rather than raw promoter/non-promoter sequences — skip feature_code.py/iLearnPlus steps
- Evaluating on independent test sets or cross-cell-line prediction rather than training-set cross-validation
- Working with non-human species where the human word2vec benchmark files (index_promoters.txt, word2vec_promoters.txt) do not apply

## Inputs

- Labeled promoter/non-promoter training sequences per cell line from the EPdata folder (first half labeled 1, second half labeled 0)
- Benchmark files index_promoters.txt and word2vec_promoters.txt for human word2vec feature extraction
- Optional example feature file R.capsulatus.rar for verifying model performance when the human feature file is unavailable

## Outputs

- Per-fold and averaged sensitivity, specificity, accuracy and MCC for each cell line across the ten folds (e.g. written to cv_results.csv)
- Trained random forest and CNN base models combined via the weighted average ensemble

## How to apply

Load labeled promoter/non-promoter sequences from the EPdata folder of the iPro-WAEL repository (first half of each file labeled 1, second half labeled 0). Extract word2vec-based features by running 'python feature_code.py' with the human benchmark files index_promoters.txt and word2vec_promoters.txt, and extract additional sequence features using iLearnPlus. Train the random forest base model with RF.py and the CNN base model with CNN.py (keras 2.3.1 / tensorflow 2.0.0 backend), then execute 'python main.py', which imports RF, CNN and Weighted_average, performing ten-fold cross-validation in which the two base models are combined via the weighted average algorithm. The 'cell_lines' variable must be manually modified to switch between GM12878, K562, HeLa-S3 and HUVEC. Rationale: iPro-WAEL parameters are tuned on GM12878 only, so cross-validation on all four human cell line training sets tests whether performance generalizes without retuning.

## Related tools

- **iPro-WAEL** (weighted average ensemble learning framework orchestrating the cross-validation) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **keras 2.3.1** (backend for the CNN base model (CNN.py))
- **tensorflow 2.0.0** (backend for the CNN base model (CNN.py))
- **sklearn** (random forest base model (RF.py) and cross-validation splitting)
- **iLearnPlus** (extraction of additional sequence features beyond word2vec)
- **Python 3.6** (runtime for feature_code.py, RF.py, CNN.py, Weighted_average.py and main.py)

## Examples

```
python main.py  # with cell_lines set in turn to GM12878, K562, HeLa-S3, HUVEC; preceded by python feature_code.py
```

## Evaluation signals

- Reproduced per-cell-line sensitivity, specificity, accuracy and MCC match the extremely high performance values reported in Supplementary Table S5 within a small tolerance
- The model with parameters tuned only on GM12878 also performs excellently on K562, HeLa-S3 and HUVEC training sets without retuning
- Each EPdata training file splits correctly: first half labeled 1 (promoters), second half labeled 0 (non-promoters)
- main.py successfully imports RF, CNN and Weighted_average and runs ten folds for each value of the 'cell_lines' variable
- Metrics are recorded for all four cell lines (GM12878, K562, HeLa-S3, HUVEC) across the ten folds

## Limitations

- The feature file for the human dataset is too large to upload to GitHub; only R.capsulatus.rar is provided as an example, so human word2vec features must be regenerated with feature_code.py
- Versions are not specified for sklearn, numpy and h5py (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0), which may affect reproducibility
- The 'cell_lines' variable must be manually modified in main.py to change the predicted cell line; there is no command-line option

## Evidence
