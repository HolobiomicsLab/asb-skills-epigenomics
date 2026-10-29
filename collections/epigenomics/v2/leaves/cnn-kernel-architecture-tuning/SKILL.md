---
name: cnn-kernel-architecture-tuning
description: 'Use when tuning a Keras/TensorFlow 1D-CNN that classifies promoter sequences:
  you need to sweep learning rate, kernel count, and kernel size and select the -performing
  configuration.'
when_to_use_negative:
- You are tuning the random forest component instead (tree-number sweep is handled
  by RF.py, not CNN.py)
- You only need to run inference with the already-trained CNN models in the 'model'
  folder rather than retrain or retune
- The target dataset is a non-human species whose training sequences are not in EPdata
  for the chosen cell line
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: iPro-WAEL
  role: Framework whose CNN component is tuned; source code and EPdata training sets
    are in the repository
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: keras
  role: Keras 2.3.1 used to define and train the CNN architecture during the grid
    search
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: tensorflow
  role: TensorFlow 2.0.0 backend executing the CNN training sweeps
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: Python
  role: Python 3.6 runtime for running CNN.py and the grid search
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: h5py
  role: Handles h5 model/feature file I/O during training
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: iLearnPlus
  role: Extracts non-word2vec sequence features fed to the CNN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:ddefb53c9edbc69b5c91360782257c4e5ed345b63695b1f4796e21554f977b05
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
  - h5py
  - iLearnPlus
  - iPro-WAEL
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# cnn-kernel-architecture-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Grid-search tuning of CNN architecture hyperparameters (learning rate, kernel number, kernel size) for the iPro-WAEL CNN promoter-identification component, validated on the GM12878 cell line dataset.

## When to use

Use when tuning a Keras/TensorFlow 1D-CNN that classifies promoter sequences: you need to sweep learning rate, kernel count, and kernel size and select the best-performing configuration.

## When NOT to use

- You are tuning the random forest component instead (tree-number sweep is handled by RF.py, not CNN.py)
- You only need to run inference with the already-trained CNN models in the 'model' folder rather than retrain or retune
- The target dataset is a non-human species whose training sequences are not in EPdata for the chosen cell line

## Inputs

- Promoter training sequences for a cell line from the EPdata folder (positive and negative halves labeled 1 and 0)
- Extracted feature matrices (word2vec via feature_code.py; other features via iLearnPlus)
- Candidate hyperparameter grid: learning rates, kernel numbers, kernel sizes

## Outputs

- grid_search_results.csv containing sweep metrics and selected optima
- Best CNN configuration (e.g. learning rate 0.001, 32 kernels, kernel size 11 on GM12878)
- Trained CNN model usable in Weighted_average.py ensemble

## How to apply

Load promoter training sequences for the target cell line (e.g. GM12878) from the EPdata folder, where each file's first half is labeled 1 and second half labeled 0, and extract features using feature_code.py (word2vec) and iLearnPlus. Train CNN models with CNN.py in Keras/TensorFlow over a grid of learning rate, kernel number, and kernel size values, recording validation performance for each configuration. For iPro-WAEL on GM12878, the grid search identified learning rate 0.001, 32 kernels, and kernel size 11 as optimal, and these parameters were adopted for the ensemble's CNN component. Select the configuration with the best cross-validated/validation metric and write the full sweep plus the optima to grid_search_results.csv.

## Related tools

- **iPro-WAEL** (Framework whose CNN component is tuned; source code and EPdata training sets are in the repository) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **keras** (Keras 2.3.1 used to define and train the CNN architecture during the grid search) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **tensorflow** (TensorFlow 2.0.0 backend executing the CNN training sweeps) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **Python** (Python 3.6 runtime for running CNN.py and the grid search) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **h5py** (Handles h5 model/feature file I/O during training) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **iLearnPlus** (Extracts non-word2vec sequence features fed to the CNN)

## Examples

```
python CNN.py  # train CNN over learning rate / kernel number / kernel size grid; optima (lr=0.001, 32 kernels, kernel size 11) written to grid_search_results.csv
```

## Evaluation signals

- Reproduced optimum on GM12878 must be learning rate 0.001, 32 kernels, kernel size 11 (matches the reported values)
- grid_search_results.csv contains recorded validation metrics for every configuration in the sweep, not only the winner
- Training data split from EPdata respects the convention: first half of each file labeled 1, second half labeled 0
- Trained CNN integrates correctly into Weighted_average.py without retuning, matching main.py behavior
- Environment versions match the paper's stack (Python 3.6, keras 2.3.1, tensorflow 2.0.0)

## Limitations

- The feature file for the human dataset is too large to upload to GitHub; only R.capsulatus.rar is provided as an example feature file
- Versions are not specified for sklearn, numpy, and h5py (unlike Python, keras, and tensorflow), which may affect reproducibility
- Optimal hyperparameters were identified on GM12878; the paper compares across seven species without retuning parameters, so transfers to other cell lines/species may not hold

## Evidence

- [other] Grid search on GM12878 identified an RF model with 300 trees as best, and a CNN model with learning rate 0.001, 32 kernels, and kernel size 11 as best; these parameters were adopted for both models.: [evidence span withheld — anchor status: not_found]
- [methods] CNN.py is the code of the CNN model.: "The file "CNN.py" is the code of the CNN model."
- [readme] EPdata folder contains enhancer/promoter sequences with first half labeled 1 and second half labeled 0.: "The folder "EPdata" contains the data of the enhancer and promoter, containing the sequences of the independent test sets and training sets. The first half of each file is labeled 1, and the second"
- [readme] Dependency list pinning keras 2.3.1 and tensorflow 2.0.0.: "Python 3.6
keras  2.3.1
tensorflow 2.0.0"
- [methods] The extraction of other features is done using iLearnPlus.: "The extraction of other features is done using iLearnPlus [1]."
