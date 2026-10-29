---
name: ensemble-weight-optimization-slsqp
description: Use when fusing probabilistic outputs of two heterogeneous base classifiers
  (e.g., RF on encoded features and CNN on word2vec features) into a single weighted-average
  ensemble and the optimal weights must be learned rather than hand-tuned.
when_to_use_negative:
- You have a single base model and no ensemble combination to optimize (e.g., only
  RF.py or only CNN.py predictions)
- The base model outputs are hard labels rather than continuous predicted values,
  making a weighted average and cross-entropy minimization ill-defined
- No dedicated weighted set with true labels is available to estimate the cross-entropy
  objective
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: iPro-WAEL
  role: Framework whose Weighted_average.py implements the SLSQP cross-entropy weight
    optimization; main.py imports RF, CNN and Weighted_average
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: sklearn
  role: Trains the random forest base classifier whose outputs feed the weighted average
- name: keras 2.3.1
  role: Trains the CNN base classifier on word2vec promoter features whose outputs
    feed the weighted average
- name: tensorflow 2.0.0
  role: Backend for the Keras CNN base model
- name: numpy
  role: Numerical operations for the weighted-average combination and loss computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:0490f3739cf7e326a3fcc2de108cae5824ec514ccae395f5ca730d07dec7f213
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
  - keras 2.3.1
  - numpy
  - sklearn
  - tensorflow
  - tensorflow 2.0.0
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# ensemble-weight-optimization-slsqp

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimizes the weighted-average combination of a random forest (RF) and a CNN classifier by minimizing cross-entropy loss on a dedicated weighted set via SLSQP, as implemented in iPro-WAEL's Weighted_average.py for promoter prediction across seven species.

## When to use

Use when fusing probabilistic outputs of two heterogeneous base classifiers (e.g., RF on encoded features and CNN on word2vec features) into a single weighted-average ensemble and the optimal weights must be learned rather than hand-tuned.

## When NOT to use

- You have a single base model and no ensemble combination to optimize (e.g., only RF.py or only CNN.py predictions)
- The base model outputs are hard labels rather than continuous predicted values, making a weighted average and cross-entropy minimization ill-defined
- No dedicated weighted set with true labels is available to estimate the cross-entropy objective

## Inputs

- RF predicted values (scikit-learn random forest probabilities) on the weighted set
- CNN predicted values (Keras/TensorFlow model probabilities) on the weighted set
- True binary labels of the weighted set (0/1 promoter labels)
- word2vec features from feature_code.py and RCKmer/Mismatch/CKSNAP/TPCP/PseTNC encodings from iLearnPlus used to train the base models

## Outputs

- Optimized ensemble weight vector for the RF and CNN combination (Weighted_average.py)
- Final iPro-WAEL weighted-average promoter predictions produced by main.py

## How to apply

Train the RF classifier (RF.py, scikit-learn) on RCKmer/Mismatch/CKSNAP/TPCP/PseTNC feature vectors generated with iLearnPlus, and train the CNN (CNN.py, Keras 2.3.1 on TensorFlow 2.0.0) on word2vec promoter features extracted with feature_code.py. Then, in Weighted_average.py, define the ensemble prediction as a weighted average of the RF and CNN predicted values and use SLSQP (scipy.optimize) to find the weights that minimize the cross-entropy loss between the ensemble predictions and true labels on a dedicated weighted set. Finally, run main.py, which imports RF, CNN and Weighted_average, to produce the final iPro-WAEL promoter predictions. The rationale is that a learned weighted average balances the complementary error profiles of the two models instead of fixing arbitrary combination coefficients.

## Related tools

- **iPro-WAEL** (Framework whose Weighted_average.py implements the SLSQP cross-entropy weight optimization; main.py imports RF, CNN and Weighted_average) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **sklearn** (Trains the random forest base classifier whose outputs feed the weighted average)
- **keras 2.3.1** (Trains the CNN base classifier on word2vec promoter features whose outputs feed the weighted average)
- **tensorflow 2.0.0** (Backend for the Keras CNN base model)
- **numpy** (Numerical operations for the weighted-average combination and loss computation)

## Examples

```
python main.py
```

## Evaluation signals

- Weighted_average.py converges under SLSQP and returns weights that reduce cross-entropy on the weighted set relative to unweighted (0.5/0.5) averaging
- Running main.py end-to-end produces final iPro-WAEL promoter predictions without import errors across RF, CNN and Weighted_average modules
- The pipeline can be verified on the R.capsulatus.rar example feature file, which ships in the repository for model-performance verification
- Ensemble predictions on the independent test sets in EPdata should outperform either the RF or CNN alone, consistent with iPro-WAEL's reported superiority over eight existing promoter prediction models

## Limitations

- The optimized weights are dataset- and species-specific; the repository notes that the 'cell_lines' variable must be manually modified to change the predicted cell line
- The human dataset feature file is too large to upload to GitHub, so weight optimization for human must be reproduced locally
- Versions are only pinned for Python 3.6, keras 2.3.1 and tensorflow 2.0.0; sklearn, numpy and h5py versions are unspecified (see requirements.txt)

## Evidence

- [other] In iPro-WAEL, the optimal ensemble weights are obtained by minimizing the cross-entropy loss between predicted values and true labels on a dedicated weighted set.: [evidence span withheld — anchor status: not_found]
