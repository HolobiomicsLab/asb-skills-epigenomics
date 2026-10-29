---
name: five-fold-iterative-cross-validation
description: Use when training/evaluating a cell-specific enhancer predictor on the
  Enhancer-IF benchmark (eight human cell lines, per-cell-line train FASTA plus independent
  test FASTA) and you need cross-validated training with head-to-head comparison on
  five metrics.
when_to_use_negative:
- The task is only feature extraction (running dna2vec_code.py or motif_find.py) rather
  than model training and test-set evaluation — cross-validation is not involved.
- The goal is cross-cell-line enhancer prediction via transfer learning or model interpretation
  of TFBS motifs (SHAP.py), rather than within-cell-line training and benchmark evaluation.
- Input sequences are not cell-specific enhancer/promoter data organized in the 'data/train'
  and 'data/test' layout with matching label .txt files.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Enhancer-MDLF
  role: The multi-input deep learning model trained under the five-fold iterative
    training strategy with focal loss to predict cell-specific enhancers
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow
  role: Deep learning framework (v2.2.0) on which main.py compiles and runs the Enhancer-MDLF
    network
- name: scikit-learn
  role: Used in the Python 3.8 environment to support model evaluation during training
    and test prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:c05c934af83ea130c55a1c3d4c6e643bebf9d0d46c693b382bed6a863b6c7c3a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  repo_url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  tools:
  - Enhancer-MDLF
  - Python
  - numpy
  - pip
  - scikit-learn
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# five-fold-iterative-cross-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Training and evaluation protocol for Enhancer-MDLF in which the multi-input deep learning model (dna2vec features + HOCOMOCO v11 motif features, optimized with focal loss) is trained under a five-fold iterative training strategy per cell line, then evaluated on the independent test set of the Enhancer-IF benchmark. It exists to give a robust, comparable estimate of cell-specific enhancer prediction performance against Enhancer-IF.

## When to use

Use when training/evaluating a cell-specific enhancer predictor on the Enhancer-IF benchmark (eight human cell lines, per-cell-line train FASTA plus independent test FASTA) and you need cross-validated training with head-to-head comparison on five metrics.

## When NOT to use

- The task is only feature extraction (running dna2vec_code.py or motif_find.py) rather than model training and test-set evaluation — cross-validation is not involved.
- The goal is cross-cell-line enhancer prediction via transfer learning or model interpretation of TFBS motifs (SHAP.py), rather than within-cell-line training and benchmark evaluation.
- Input sequences are not cell-specific enhancer/promoter data organized in the 'data/train' and 'data/test' layout with matching label .txt files.

## Inputs

- Per-cell-line training FASTA sequences (e.g. 'data/train/NHLF.fasta') with integer label files ('data/train/${cell_line}_y_train.txt')
- Per-cell-line independent test FASTA sequences ('data/test/${cell_line}.fasta') with label files ('data/test/${cell_line}_y_test.txt')
- Pre-extracted dna2vec and motif features for train and test splits, written to the 'feature' directory
- Pre-trained dna2vec DNA vectors and HOCOMOCO Human v11 PWMs with p-value threshold scores
- Enhancer-IF reported benchmark results per cell line for comparison

## Outputs

- Trained Enhancer-MDLF models per cell line (saved under the 'model' folder; pre-trained models on all cell lines are also provided)
- Predictions on each cell line's independent test set
- Per-cell-line values of the five evaluation metrics (including BACC), tabulated against Enhancer-IF

## How to apply

First extract dna2vec-based features and motif-based features (using pre-trained dna2vec vectors and HOCOMOCO Human v11 PWMs with their p-value thresholds) into the 'feature' directory, since training cannot start without both feature views. Partition the training sequences for the target cell line (e.g. 'data/train/NHLF.fasta' with 'NHLF_y_train.txt') into five folds and run the iterative training strategy: repeatedly train the Enhancer-MDLF network with focal loss on four folds and validate on the held-out fold, mitigating the class imbalance inherent in enhancer data. After the five-fold iterations, compile and run the full model via main.py with the --cell_line argument to predict on the independent test set ('data/test/${cell_line}.fasta' with '${cell_line}_y_test.txt'), which must remain untouched during fold training. Tabulate the five evaluation metrics (including BACC) per cell line against the reported Enhancer-IF results to judge whether the model was trained and evaluated correctly.

## Related tools

- **Enhancer-MDLF** (The multi-input deep learning model trained under the five-fold iterative training strategy with focal loss to predict cell-specific enhancers) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow** (Deep learning framework (v2.2.0) on which main.py compiles and runs the Enhancer-MDLF network)
- **scikit-learn** (Used in the Python 3.8 environment to support model evaluation during training and test prediction)

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- On the independent test sets across all eight cell lines, the trained model consistently outperforms Enhancer-IF on all five evaluation metrics.
- The BACC improvement on the HMEC cell line is approximately 9.54% over Enhancer-IF.
- Per-cell-line metrics are tabulated against the reported Enhancer-IF results and match the benchmark direction (Enhancer-MDLF superior) on every cell line.
- Both dna2vec and motif feature files exist in the 'feature' directory for the requested --set (train or test) before main.py is launched.
- The independent test labels ('${cell_line}_y_test.txt') are never used during the five-fold training iterations.

## Limitations

- The protocol is tied to the Enhancer-IF benchmark layout and its eight human cell lines; other datasets must be reorganized into the same train/test FASTA-plus-label format.
- Requires the pinned environment (Python 3.8, pip 23.3.1, tensorflow 2.2.0, scikit-learn, numpy) installed via requirements.txt; version conflicts break main.py.
- Performance claims depend on the provided independent test sets; results are only meaningful when those sets are kept held-out from fold training.

## Evidence

- [methods] Train Enhancer-MDLF with focal loss under the five-fold iterative training strategy and predict on the independent test sets: [evidence span withheld — anchor status: not_found]
- [readme] Users can run the script as follows to compile and run Enhancer-MDLF:    `python main.py --cell_line ${cell_line}`: [evidence span withheld — anchor status: not_found]
- [results] Enhancer-MDLF consistently outperforms Enhancer-IF on all five evaluation metrics, including a 9.54% BACC improvement over Enhancer-IF on the HMEC cell line.: [evidence span withheld — anchor status: not_found]
