---
name: tree-ensemble-classifier-training
description: Use when several candidate classifiers must be compared on a fixed feature
  matrix (e.g., Motif+Kmer+Binary encoded DNA 6mA sequences) via cross-validation
  to select the algorithm before final model training.
when_to_use_negative:
- Input is already a trained model or prediction score file — use the independent
  testing script (csDMA_indepedent_testing.py) instead of retraining
- The goal is evaluation only (e.g., LeaveOneOut scoring) — use leaveoneout.py rather
  than re-running classifier selection
- The feature matrix is not yet encoded from raw sequences — run the feature encoding
  scheme (feature_selection.py) first
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: csDMA
  role: Python 2.7 pipeline containing classifier_selection.py (trains the five candidate
    classifiers with 5-fold CV) and csDMA_training.py (trains the final ExtraTrees
    model producing csDMA.pkl and the normalization file)
  repo: https://github.com/liuze-nwafu/csDMA
- name: ExtraTrees
  role: The selected best classifier (1,000 trees) used to train the optimized csDMA
    model after cross-validated comparison
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:3fd6727432701367b38cf19ce4efc71db76e683b3b7c64ffe8106ea0dc4a8452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - ExtraTrees
  - Python
  - csDMA
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# tree-ensemble-classifier-training

## Summary

Train and compare tree-ensemble classifiers (Random Forest, GradientBoosting, AdaBoost, ExtraTrees) on encoded sequence feature matrices to select the best-performing model, as done in csDMA where ExtraTrees achieved the best AUC of 0.878 by 5-fold cross-validation and was used to train the final model.

## When to use

Use when several candidate classifiers must be compared on a fixed feature matrix (e.g., Motif+Kmer+Binary encoded DNA 6mA sequences) via cross-validation to select the best algorithm before final model training.

## When NOT to use

- Input is already a trained model or prediction score file — use the independent testing script (csDMA_indepedent_testing.py) instead of retraining
- The goal is evaluation only (e.g., LeaveOneOut scoring) — use leaveoneout.py rather than re-running classifier selection
- The feature matrix is not yet encoded from raw sequences — run the feature encoding scheme (feature_selection.py) first

## Inputs

- Training positive sequence dataset (6mA-modified, FASTA-like)
- Training negative sequence dataset (unmodified)
- Encoded motif feature matrices (matrix_motif1, matrix_motif2) produced by feature_selection.py
- Motif+Kmer+Binary feature matrix (k-mer k=2,3,4; 336 k-mer features plus binary and motif scores)

## Outputs

- Per-classifier 5-fold cross-validation metrics table (Sn, Sp, ACC, MCC, AUC, F1), e.g., classifier_cv_metrics.csv
- Selected best classifier decision (ExtraTrees in csDMA)
- Trained model file csDMA.pkl (ExtraTrees-based)
- Normalization/scale file (scale_file / normalization.pkl) for feature scaling before prediction

## How to apply

Assemble the encoded feature matrix (in csDMA: Motif, Kmer with k=2,3,4 yielding 336 k-mer features, and Binary subsets) from positive and negative training sequence datasets. Train each candidate classifier — Random Forest, GradientBoosting, AdaBoost, and ExtraTrees each configured with 1,000 trees, plus an SVM with grid-searched C and gamma — inside the csDMA Python 2.7 pipeline using classifier_selection.py. Evaluate all candidates with 5-fold cross-validation, computing Sn, Sp, ACC, MCC, AUC, and F1 for each. Select the classifier with the best metrics (csDMA selected ExtraTrees: MCC 0.603, AUC 0.878, F1 0.811 on training; Sn 0.888, AUC 0.893, F1 0.832 on independent testing) and train the final model with csDMA_training.py, which outputs the model file csDMA.pkl and the normalization file scale_file (normalization.pkl) required for later prediction.

## Related tools

- **csDMA** (Python 2.7 pipeline containing classifier_selection.py (trains the five candidate classifiers with 5-fold CV) and csDMA_training.py (trains the final ExtraTrees model producing csDMA.pkl and the normalization file)) — https://github.com/liuze-nwafu/csDMA
- **ExtraTrees** (The selected best classifier (1,000 trees) used to train the optimized csDMA model after cross-validated comparison) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- 5-fold cross-validation metrics for ExtraTrees should reach approximately Sn 0.864, ACC 0.799, MCC 0.603, AUC 0.878, F1 0.811 on the training dataset (Table 2)
- On the independent testing dataset, ExtraTrees should achieve approximately Sn 0.888, AUC 0.893, F1 0.832, while SVM reaches the best Sp of 0.761 (Table 3)
- csDMA_training.py must output both a model file (csDMA.pkl) and a normalized file (normalization.pkl); prediction requires both
- The cross-validation comparison table (classifier_cv_metrics.csv) must contain Sn, ACC, MCC, AUC and F1 for all five classifiers, with ExtraTrees ranked best by AUC
- Independent testing must reuse the scale_file generated during training for feature normalization before prediction

## Limitations

- csDMA was implemented in Python 2.7; the pipeline may require a legacy environment
- csDMA performance was relatively low on the rice and cross-species datasets compared to the M. musculus dataset
- The classifier_selection.py script also supports a sixth ensemble classifier combining the five methods, but csDMA itself was trained with ExtraTrees only
- SVM hyperparameters (penalty C and kernel coefficient gamma) require grid search, adding tuning cost relative to the tree ensembles

## Evidence

- [readme] classifier selection: "This script trains five different 6mA prediction models.
Including:
     1--RandomForest
     2--GradientBoosting
     3--AdaBoost
     4--ExtraTrees
     5--SVM"
- [readme] cross-validation evaluation: "5-fold cross-validation was used to evaluate the performance of different classifier."
- [intro] ExtraTrees selection: "Thus, we used the ExtraTrees algorithm to train the optimized model."
- [readme] training outputs: "1--a model file, csDMA.pkl, which can be directly used for prediction."
- [abstract] best CV performance: "the ExtraTrees model received the best AUC of 0.878 by using 5-fold cross-validation on the training dataset"
