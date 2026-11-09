---
name: cross-species-promoter-model-validation
description: 'Use when you must reproduce or extend iPro-WAEL''s promoter-vs-enhancer
  discrimination benchmark: enhancer sequences as negatives, subsampled to match promoter
  counts, with model parameters kept unchanged.'
when_to_use_negative:
- Input is already a feature table (e.g. the provided R.capsulatus.rar feature file)
  — skip feature_code.py/iLearnPlus extraction and run main.py directly for model
  verification
- The goal is de novo promoter prediction across species rather than promoter-vs-enhancer
  discrimination on the balanced EPdata benchmark
- You need retuned or cell-line-specific hyperparameters — this skill explicitly reuses
  iPro-WAEL's parameters unchanged per the experiment design
edam_topics:
- http://edamontology.org/topic_3511
tools:
- name: iPro-WAEL
  role: Weighted average ensemble framework (main.py) that combines the RF and CNN
    base models to produce final promoter/enhancer predictions
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: feature_code.py
  role: Extracts the word2vec-based features used to encode EPdata enhancer/promoter
    sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: iLearnPlus
  role: Extracts the remaining (non-word2vec) sequence features from the EPdata sequences
- name: RF.py
  role: scikit-learn random forest base model trained on the encoded features without
    retuning
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: CNN.py
  role: keras 2.3.1 / tensorflow 2.0.0 CNN base model trained on the encoded features
    without retuning
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: Weighted_average.py
  role: Weighted average algorithm invoked via main.py to combine RF and CNN outputs
    into final predictions
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:e91f474d7d02c61b5f8d9893441cc819d7c9cc8385ace2082646ea42754128e4
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
  - CNN.py
  - Python
  - RF.py
  - Weighted_average.py
  - feature_code.py
  - h5py
  - iLearnPlus
  - iPro-WAEL
  - keras
  - numpy
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# cross-species-promoter-model-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the iPro-WAEL promoter-versus-enhancer discrimination experiment: the pre-trained RF and CNN base models are run with unchanged parameters on balanced EPdata sequences, and their weighted-average ensemble predictions are evaluated against Supplementary Table S19.

## When to use

Use when you must reproduce or extend iPro-WAEL's promoter-vs-enhancer discrimination benchmark: enhancer sequences as negatives, subsampled to match promoter counts, with model parameters kept unchanged.

## When NOT to use

- Input is already a feature table (e.g. the provided R.capsulatus.rar feature file) — skip feature_code.py/iLearnPlus extraction and run main.py directly for model verification
- The goal is de novo promoter prediction across species rather than promoter-vs-enhancer discrimination on the balanced EPdata benchmark
- You need retuned or cell-line-specific hyperparameters — this skill explicitly reuses iPro-WAEL's parameters unchanged per the experiment design

## Inputs

- Enhancer and promoter sequence files from the 'EPdata' folder (independent test sets and training sets; first half of each file = label 1, second half = label 0)
- Target cell line name (set in the 'cell_lines' variable of main.py)
- word2vec benchmark files index_promoters.txt and word2vec_promoters.txt for human feature extraction

## Outputs

- promoter/enhancer predictions from the weighted average ensemble of RF and CNN outputs
- performance_metrics.csv with discrimination metrics computed on the independent test set
- word2vec-based feature encodings plus iLearnPlus-extracted sequence features

## How to apply

Load enhancer and promoter sequences from the 'EPdata' folder, treating the first half of each file as positives (label 1) and the second half as negatives (label 0), then subsample so positive and negative counts are equal per cell line. Encode the sequences into word2vec-based features by running feature_code.py, and extract the remaining sequence features with iLearnPlus. Manually set the 'cell_lines' variable in main.py to the target cell line, then run the random forest base model (RF.py, scikit-learn) and the CNN base model (CNN.py, keras 2.3.1 with tensorflow 2.0.0) on the encoded features without retuning parameters. Combine the RF and CNN outputs with the weighted average algorithm (Weighted_average.py) invoked through main.py, compute discrimination metrics on the independent test set, write them to performance_metrics.csv, and verify agreement with the promoter-vs-enhancer performance in Supplementary Table S19.

## Related tools

- **iPro-WAEL** (Weighted average ensemble framework (main.py) that combines the RF and CNN base models to produce final promoter/enhancer predictions) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **feature_code.py** (Extracts the word2vec-based features used to encode EPdata enhancer/promoter sequences) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **iLearnPlus** (Extracts the remaining (non-word2vec) sequence features from the EPdata sequences)
- **RF.py** (scikit-learn random forest base model trained on the encoded features without retuning) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **CNN.py** (keras 2.3.1 / tensorflow 2.0.0 CNN base model trained on the encoded features without retuning) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **Weighted_average.py** (Weighted average algorithm invoked via main.py to combine RF and CNN outputs into final predictions) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL

## Examples

```
python feature_code.py && python main.py
```

## Evaluation signals

- performance_metrics.csv is produced and its independent-test-set metrics agree with the promoter-vs-enhancer performance reported in Supplementary Table S19
- Positive and negative counts are equal per cell line after subsampling enhancer negatives to match promoter counts
- Label assignment follows the EPdata convention: first half of each file labeled 1, second half labeled 0
- RF.py and CNN.py are executed with unchanged (un-retuned) parameters, matching the published experiment design
- Final predictions equal the weighted average of the RF and CNN outputs rather than either base model alone

## Limitations

- The feature file for the human dataset is too large to be hosted on GitHub, so features must be re-extracted locally (only R.capsulatus.rar is provided as an example feature file)
- The 'cell_lines' variable in main.py must be manually modified to change the predicted cell line — there is no CLI parameter for it
- Versions are pinned only for Python 3.6, keras 2.3.1 and tensorflow 2.0.0; versions for sklearn, numpy and h5py are unspecified, so environment reproducibility depends on requirements.txt

## Evidence
