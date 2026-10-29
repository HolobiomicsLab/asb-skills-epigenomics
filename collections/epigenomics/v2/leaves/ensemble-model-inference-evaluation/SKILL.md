---
name: ensemble-model-inference-evaluation
description: Use when labeled DNA promoter/enhancer sequences must be classified by
  a pre-trained ensemble without retuning, e.g. cross-cell-line prediction or promoter-vs-enhancer
  discrimination on balanced datasets.
when_to_use_negative:
- Input is already a feature table or raw model scores rather than DNA sequences needing
  word2vec/iLearnPlus encoding
- The task is motif/TFBS interpretation rather than ensemble inference — use feature_importance.rar
  instead of RF.py/CNN.py/Weighted_average.py
- Sequences are from species outside the seven covered by the 12 trained datasets
  in the 'model' folder
edam_operation: http://edamontology.org/operation_0465
edam_topics:
- http://edamontology.org/topic_3510
tools:
- name: iPro-WAEL
  role: The weighted average ensemble learning framework being run and evaluated;
    orchestrated through main.py
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: keras 2.3.1
  role: Framework for the CNN base model (CNN.py) applied to encoded features
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: tensorflow 2.0.0
  role: Backend for the keras CNN base model
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: sklearn
  role: Implements the random forest base model (RF.py)
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: feature_code.py
  role: Extracts word2vec-based features of promoters prior to ensemble inference
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: iLearnPlus
  role: Extracts the remaining (non-word2vec) sequence features fed to the ensemble
- name: Weighted_average.py
  role: Combines RF and CNN outputs into final ensemble predictions
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: h5py
  role: Dependency for handling model/feature files during inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:56c33ed873324b600c4c782104869b2f28b9fa3c20c76726dd5a9e15279b2acb
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
  - Weighted_average.py
  - feature_code.py
  - h5py
  - iLearnPlus
  - iPro-WAEL
  - keras
  - keras 2.3.1
  - numpy
  - sklearn
  - tensorflow 2.0.0
  edam_operation: http://edamontology.org/operation_0465
  edam_topics:
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# ensemble-model-inference-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run and evaluate the iPro-WAEL weighted average ensemble — a random forest (RF.py, scikit-learn) combined with a CNN (CNN.py, keras 2.3.1/tensorflow 2.0.0) via Weighted_average.py — over word2vec and iLearnPlus-encoded sequence features to produce promoter predictions and discrimination metrics.

## When to use

Use when labeled DNA promoter/enhancer sequences must be classified by a pre-trained ensemble without retuning, e.g. cross-cell-line prediction or promoter-vs-enhancer discrimination on balanced datasets.

## When NOT to use

- Input is already a feature table or raw model scores rather than DNA sequences needing word2vec/iLearnPlus encoding
- The task is motif/TFBS interpretation rather than ensemble inference — use feature_importance.rar instead of RF.py/CNN.py/Weighted_average.py
- Sequences are from species outside the seven covered by the 12 trained datasets in the 'model' folder

## Inputs

- Promoter sequences from the 'data' folder (12 datasets, seven species: Human, Mouse, E.coli, Arabidopsis, B.amyloliquefaciens, B.subtilis, R.capsulatus)
- Enhancer and promoter sequences from the 'EPdata' folder (first half of each file labeled 1, second half labeled 0), subsampled so positive and negative counts are equal per cell line
- Pre-trained models in the 'model' folder (trained on 12 datasets of seven species)
- Feature files (e.g. R.capsulatus.rar as an example verification feature file)

## Outputs

- Final promoter/enhancer class predictions from the weighted average ensemble
- Discrimination metrics on the independent test set written to performance_metrics.csv
- Trained RF.py and CNN.py base-model outputs (e.g. .h5/model files) used by Weighted_average.py

## How to apply

Encode sequences with word2vec-based features using feature_code.py and extract remaining features with iLearnPlus. Manually set the 'cell_lines' variable in main.py to the target cell line, then run the random forest base model (RF.py, scikit-learn) and the CNN base model (CNN.py, keras 2.3.1 with tensorflow 2.0.0) on the encoded features without retuning parameters. Combine RF and CNN outputs with the weighted average algorithm (Weighted_average.py) invoked through main.py to obtain final predictions. Evaluate on the independent test set and compare metrics (e.g. promoter-vs-enhancer performance in Supplementary Table S19) to verify the ensemble reproduces reported performance, since iPro-WAEL is reported to be superior to current methods across 12 datasets of seven species.

## Related tools

- **iPro-WAEL** (The weighted average ensemble learning framework being run and evaluated; orchestrated through main.py) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **keras 2.3.1** (Framework for the CNN base model (CNN.py) applied to encoded features) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **tensorflow 2.0.0** (Backend for the keras CNN base model) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **sklearn** (Implements the random forest base model (RF.py)) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **feature_code.py** (Extracts word2vec-based features of promoters prior to ensemble inference) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **iLearnPlus** (Extracts the remaining (non-word2vec) sequence features fed to the ensemble)
- **Weighted_average.py** (Combines RF and CNN outputs into final ensemble predictions) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **h5py** (Dependency for handling model/feature files during inference)

## Examples

```
python feature_code.py && python main.py
```

## Evaluation signals

- Metrics on the independent test set written to performance_metrics.csv agree with the promoter-vs-enhancer performance reported in Supplementary Table S19
- Positive and negative counts are equal per cell line (first half of each EPdata file labeled 1, second half labeled 0)
- RF and CNN base models run with parameters kept unchanged (no retuning) across species/cell lines
- R.capsulatus.rar example feature file reproduces the reported model performance as a verification check
- Ensemble metrics exceed the eight comparison promoter prediction methods reported in the benchmarking experiments

## Limitations

- The variable 'cell_lines' must be manually modified to change the predicted cell line; there is no automated configuration
- The feature file for the human dataset is too large to upload to GitHub, so word2vec features must be regenerated locally
- Versions are not specified for sklearn, numpy, and h5py (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0); consult requirements.txt for full dependency details
- Performance on species outside the seven trained datasets is not established; the framework is validated on 12 datasets of seven species

## Evidence
