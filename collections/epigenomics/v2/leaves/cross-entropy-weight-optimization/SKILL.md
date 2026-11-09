---
name: cross-entropy-weight-optimization
description: Use when two trained promoter classifiers (RF probability outputs and
  CNN/Keras outputs) must be fused via a weighted average whose weights are tuned
  by minimizing cross-entropy on a labeled weighted set.
when_to_use_negative:
- Only a single trained classifier is available (no RF + CNN pair to combine into
  a weighted average).
- The task is feature extraction rather than ensemble combination — use feature_code.py
  or iLearnPlus instead.
- Input is a different model family whose outputs were not produced by RF.py and CNN.py
  with the iPro-WAEL feature encodings.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: iPro-WAEL
  role: Host framework; Weighted_average.py implements the SLSQP cross-entropy weight
    optimization and main.py imports it with RF and CNN
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: Python 3.6
  role: Runtime environment in which Weighted_average.py, RF.py and CNN.py are executed
- name: sklearn
  role: Provides the random forest base learner whose probabilities are combined by
    the optimized weights
- name: keras 2.3.1
  role: Provides the CNN base learner (on TensorFlow 2.0.0) whose probabilities are
    combined by the optimized weights
- name: numpy
  role: Numerical handling of prediction vectors and weights during cross-entropy
    minimization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:e94644a4874839fb0b99aec66446d0ea4772f52073af99aeba4bc4e781fcc1d8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cross-entropy-weight-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimizes the weighted-average combination of a random forest (RF) and a CNN promoter-prediction model in iPro-WAEL by minimizing cross-entropy loss between predicted values and true labels on a dedicated weighted set, implemented in Weighted_average.py using SLSQP.

## When to use

Use when two trained promoter classifiers (RF probability outputs and CNN/Keras outputs) must be fused via a weighted average whose weights are tuned by minimizing cross-entropy on a labeled weighted set.

## When NOT to use

- Only a single trained classifier is available (no RF + CNN pair to combine into a weighted average).
- The task is feature extraction rather than ensemble combination — use feature_code.py or iLearnPlus instead.
- Input is a different model family whose outputs were not produced by RF.py and CNN.py with the iPro-WAEL feature encodings.

## Inputs

- Trained RF model outputs (probabilities) from RF.py on RCKmer/Mismatch/CKSNAP/TPCP/PseTNC feature vectors
- Trained CNN model outputs from CNN.py on word2vec promoter features
- Weighted set: labeled training/independent-test promoter sequences from the EPdata folder (first half labeled 1, second half labeled 0)

## Outputs

- Optimal ensemble combination weights for the RF and CNN models
- Final iPro-WAEL weighted-average promoter predictions via main.py

## How to apply

First train the random forest (RF.py, scikit-learn) on RCKmer/Mismatch/CKSNAP/TPCP/PseTNC feature vectors produced with iLearnPlus, and the CNN (CNN.py, Keras 2.3.1 on TensorFlow 2.0.0) on word2vec features extracted by feature_code.py. Then run Weighted_average.py, which obtains the optimal ensemble weights by minimizing the cross-entropy loss between the weighted-average predicted values and true labels on a dedicated weighted set, using SLSQP as the constrained optimizer. Finally, main.py imports RF, CNN and Weighted_average and applies the learned weights to generate the final iPro-WAEL promoter predictions. The rationale is that a supervised weight search on held-out 'weighted' data yields better ensemble calibration than naive averaging across the seven species datasets.

## Related tools

- **iPro-WAEL** (Host framework; Weighted_average.py implements the SLSQP cross-entropy weight optimization and main.py imports it with RF and CNN) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **Python 3.6** (Runtime environment in which Weighted_average.py, RF.py and CNN.py are executed)
- **sklearn** (Provides the random forest base learner whose probabilities are combined by the optimized weights)
- **keras 2.3.1** (Provides the CNN base learner (on TensorFlow 2.0.0) whose probabilities are combined by the optimized weights)
- **numpy** (Numerical handling of prediction vectors and weights during cross-entropy minimization)

## Examples

```
python main.py
```

## Evaluation signals

- Running 'python main.py' completes and imports RF, CNN and Weighted_average without error.
- The optimized weights minimize cross-entropy loss between predicted values and true labels on the dedicated weighted set.
- The full pipeline reproduces iPro-WAEL predictions when verified on the R.capsulatus.rar example feature file.
- Predictions change appropriately when the 'cell_lines' variable is manually modified to a different cell line.
- Final ensemble predictions outperform either the RF or CNN component alone on the independent test sets (first half labeled 1, second half 0).

## Limitations

- The human dataset feature file is too large to upload to GitHub, so weight optimization on human data requires regenerating word2vec features locally with feature_code.py.
- The 'cell_lines' variable must be manually modified to change the predicted cell line — no automatic per-cell-line switching.
- Versions are not specified for sklearn, numpy, and h5py (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0), so exact dependency reproduction may require requirements.txt.

## Evidence

- [readme] Weighted_average.py implements SLSQP optimization: "The file "Weighted_average.py" is the code of the weighted average algorithm."
- [readme] main.py orchestrates the ensemble: "The file "main.py" is the code of the entire model and will import RF, CNN and Weighted_average."
- [results] Cross-entropy objective on the weighted set: [evidence span withheld — anchor status: not_found]
- [methods] SLSQP optimization step in the workflow: [evidence span withheld — anchor status: not_found]
- [methods] End-to-end invocation: "Then run the script as follows to compile and run iPro-WAEL: `python main.py`"
