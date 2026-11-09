---
name: cross-cell-line-generalization-evaluation
description: Use when a trained promoter-prediction model on one human cell line must
  be evaluated on independent test sets of the other cell lines, and results compared
  to Z-curve and same-cell line baselines.
when_to_use_negative:
- The goal is within-species or cross-species promoter prediction on the 12 benchmark
  datasets of seven species rather than human cross-cell line generalization
- The input is already a computed feature matrix — this skill starts from raw promoter
  sequences and requires the feature_code.py + iLearnPlus extraction steps
- Evaluating enhancer-promoter discrimination from 'EPdata' rather than cross-cell
  line promoter prediction
edam_operation: http://edamontology.org/operation_0489
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3474
tools:
- name: iPro-WAEL
  role: Weighted average ensemble learning model (RF + CNN) whose trained cell line
    models are evaluated cross-cell line
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: Python 3.6
  role: Runtime environment for running feature_code.py, RF.py, CNN.py, Weighted_average.py
    and main.py
- name: keras 2.3.1
  role: Framework used to build and train the CNN base model (CNN.py)
- name: tensorflow 2.0.0
  role: Backend for the CNN base model training
- name: sklearn
  role: Library used for the random forest base model (RF.py) and performance metrics
- name: iLearnPlus
  role: Platform used to extract the non-word2vec promoter features
- name: numpy
  role: Numerical handling of feature arrays and model outputs
- name: h5py
  role: Storage/loading of trained model or feature data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:3bbfa440c5d4e5e77f2d77494db4298e38ef610f618a5ee874b3a836c3c6dd5d
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
  edam_operation: http://edamontology.org/operation_0489
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-cell-line-generalization-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Protocol for testing whether the iPro-WAEL promoter predictor (weighted-average ensemble of an RF model and a CNN model) trained on one human cell line retains performance on independent test sets from other cell lines, benchmarked against the previous best method (Z-curve) and against same-cell line performance.

## When to use

Use when a trained promoter-prediction model on one human cell line must be evaluated on independent test sets of the other cell lines, and results compared to Z-curve and same-cell line baselines.

## When NOT to use

- The goal is within-species or cross-species promoter prediction on the 12 benchmark datasets of seven species rather than human cross-cell line generalization
- The input is already a computed feature matrix — this skill starts from raw promoter sequences and requires the feature_code.py + iLearnPlus extraction steps
- Evaluating enhancer-promoter discrimination from 'EPdata' rather than cross-cell line promoter prediction

## Inputs

- Human promoter training and independent test sequences from the 'EPdata' folder (first half labeled 1, second half labeled 0)
- Benchmark files index_promoters.txt and word2vec_promoters.txt for word2vec feature extraction
- Trained iPro-WAEL models (RF + CNN weighted average ensemble) from the 'model' folder
- iLearnPlus-extracted feature files for promoter sequences

## Outputs

- Cross-cell line prediction performance metrics for all four human cell lines (e.g. compiled into cross_cell_line_results.csv)
- Comparison of iPro-WAEL vs. Z-curve performance and cross- vs. same-cell line performance

## How to apply

Extract word2vec-based features of the human promoter sequences by running `python feature_code.py` with the benchmark files index_promoters.txt and word2vec_promoters.txt, and extract remaining features with iLearnPlus. Load training and independent test sequences from the 'EPdata' folder (first half of each file labeled 1, second half labeled 0), then train the random forest base model (RF.py) and the CNN base model (CNN.py) with keras 2.3.1 on tensorflow 2.0.0 under Python 3.6 on each human cell line's training set. Combine RF and CNN outputs through the weighted average algorithm (Weighted_average.py) via the entire-model entry point `python main.py`. Manually modify the 'cell_lines' variable in main.py so the model trained on each cell line predicts the independent test sets of the other three cell lines, then compile cross-cell line metrics (e.g. into cross_cell_line_results.csv).

## Related tools

- **iPro-WAEL** (Weighted average ensemble learning model (RF + CNN) whose trained cell line models are evaluated cross-cell line) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **Python 3.6** (Runtime environment for running feature_code.py, RF.py, CNN.py, Weighted_average.py and main.py)
- **keras 2.3.1** (Framework used to build and train the CNN base model (CNN.py))
- **tensorflow 2.0.0** (Backend for the CNN base model training)
- **sklearn** (Library used for the random forest base model (RF.py) and performance metrics)
- **iLearnPlus** (Platform used to extract the non-word2vec promoter features)
- **numpy** (Numerical handling of feature arrays and model outputs)
- **h5py** (Storage/loading of trained model or feature data)

## Examples

```
python main.py  # with cell_lines variable in main.py manually modified to set the predicted cell line
```

## Evaluation signals

- Cross-cell line performance of iPro-WAEL should exceed the best prior method (Z-curve) on the human datasets
- Cross-cell line performance should be not significantly different from iPro-WAEL's same-cell line performance (a satisfactory result as claimed in the paper)
- The independent test sets are correctly loaded from 'EPdata' with first-half instances labeled 1 and second-half labeled 0
- Predictions are generated for all four human cell lines by cycling the 'cell_lines' variable in main.py so each cell line's model predicts the other three
- Metrics for all cell line pairs are compiled (e.g. cross_cell_line_results.csv) without missing pairwise entries

## Limitations

- The 'cell_lines' variable in main.py must be manually modified to switch the predicted cell line — cross-cell line evaluation cannot be run in a single unattended pass
- The human feature file is too large to be uploaded to GitHub, so the word2vec features must be regenerated with feature_code.py (only R.capsulatus.rar is provided as an example feature file)
- Versions are not specified for sklearn, numpy, and h5py (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0), which may complicate environment reproduction

## Evidence

- [methods] Note that the variable 'cell_lines' needs to be manually modified to change the predicted cell line.: "Note that the variable 'cell_lines' needs to be manually modified to change the predicted cell line."
- [readme] EPdata labeling: "The folder "EPdata" contains the data of the enhancer and promoter, containing the sequences of the independent test sets and training sets. The first half of each file is labeled 1, and the second"
- [abstract] cross-cell line finding: "The experimental results also demonstrate a satisfactory prediction ability of iPro-WAEL on cross-cell lines, promoters annotated by other methods and distinguishing between promoters and enhancers."
- [readme] missing human feature file: "The file "R.capsulatus.rar" is an example feature file to verify model performance (the feature file for the human dataset is too large to upload to github)."
- [methods] feature extraction entry point: "First, you should extract features of promoters, you can run the script to extract word2vec-based features as follows: `python feature_code.py`"
