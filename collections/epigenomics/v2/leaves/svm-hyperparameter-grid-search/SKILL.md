---
name: svm-hyperparameter-grid-search
description: Use when selecting among candidate classifiers (e.g., RandomForest, GradientBoosting,
  AdaBoost, ExtraTrees, SVM) on an encoded sequence feature matrix and the SVM candidate
  requires C and gamma tuning via grid search before fair cross-validated comparison.
when_to_use_negative:
- The final model has already been selected (csDMA was trained with ExtraTrees; csDMA_training.py
  does not tune SVM hyperparameters)
- The input is raw sequence without the Motif/Kmer/Binary encoding produced by feature_selection.py,
  since classifier_selection.py expects matrix_motif1/matrix_motif2 alongside the
  sequence files
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: csDMA
  role: Python 2.7 pipeline whose classifier_selection.py script trains and grid-search-tunes
    the SVM candidate against four tree ensembles with 5-fold cross-validation
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:899f7da007334b0ff0e6856e23fc7019391890257d1f7513700305ae8081f884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - Python
  - csDMA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# svm-hyperparameter-grid-search

## Summary

Grid-search tuning of SVM penalty (C) and kernel coefficient (gamma) hyperparameters, then 5-fold cross-validation evaluation, as performed in the csDMA pipeline to test whether an SVM classifier beats tree ensembles on the Motif+Kmer+Binary DNA 6mA feature matrix.

## When to use

Use when selecting among candidate classifiers (e.g., RandomForest, GradientBoosting, AdaBoost, ExtraTrees, SVM) on an encoded sequence feature matrix and the SVM candidate requires C and gamma tuning via grid search before fair cross-validated comparison.

## When NOT to use

- The final model has already been selected (csDMA was trained with ExtraTrees; csDMA_training.py does not tune SVM hyperparameters)
- The input is raw sequence without the Motif/Kmer/Binary encoding produced by feature_selection.py, since classifier_selection.py expects matrix_motif1/matrix_motif2 alongside the sequence files

## Inputs

- Training positive 6mA sequence dataset (e.g., 41nt FASTA from the M. musculus / rice / cross-species benchmark)
- Training negative non-6mA sequence dataset
- Encoded motif feature matrix matrix_motif1 (from feature_selection.py encoding scheme)
- Encoded motif feature matrix matrix_motif2

## Outputs

- Per-classifier 5-fold cross-validation metrics (Sn, ACC, MCC, AUC, F1) for the tuned SVM, comparable with the tree ensembles
- Classifier comparison table (e.g., classifier_cv_metrics.csv) supporting the final classifier choice

## How to apply

In the csDMA Python 2.7 pipeline, encode the training positive/negative 41nt 6mA sequences into the Motif+Kmer+Binary feature matrix using the feature_selection.py scheme (motif matrices matrix_motif1 and matrix_motif2 are passed separately as command-line arguments). Run classifier_selection.py, which trains the SVM alongside RandomForest, GradientBoosting, AdaBoost and ExtraTrees (the tree models use 1,000 trees), tuning the SVM penalty (C) and kernel coefficient (gamma) by grid search. Evaluate every classifier with 5-fold cross-validation, computing Sn, ACC, MCC, AUC and F1 for each. Treat the tuned SVM as one candidate among several: in this study it underperformed ExtraTrees (AUC 0.878, MCC 0.603, F1 0.811), so ExtraTrees was chosen for the final csDMA model.

## Related tools

- **csDMA** (Python 2.7 pipeline whose classifier_selection.py script trains and grid-search-tunes the SVM candidate against four tree ensembles with 5-fold cross-validation) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- The tuned SVM's 5-fold CV metrics (Sn, ACC, MCC, AUC, F1) are reported alongside all five classifiers, not in isolation
- SVM performance must be compared against ExtraTrees' reported training-set benchmarks (AUC 0.878, Sn 0.864, ACC 0.799, MCC 0.603, F1 0.811) to justify the final classifier choice
- classifier_selection.py runs on exactly four inputs: training positive dataset, training negative dataset, matrix_motif1, matrix_motif2
- The same encoded Motif+Kmer+Binary feature matrix is used for every classifier so the comparison is not confounded by feature subsets

## Limitations

- The SVM was not selected: ExtraTrees achieved the best AUC of 0.878 by 5-fold cross-validation on the training dataset, so tuned-SVM results do not carry forward into csDMA.pkl
- Grid search covers only the penalty (C) and kernel coefficient (gamma) hyperparameters as described; other SVM design choices (kernel type, scaling) are not specified in the context
- csDMA was implemented in Python 2.7, so the script may require a legacy environment

## Evidence

- [readme] classifier selection usage: "Usage: python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2"
- [readme] five classifiers and CV: [evidence span withheld — anchor status: not_found]
- [intro] ExtraTrees selected over candidates: "Thus, we used the ExtraTrees algorithm to train the optimized model."
- [discussion] CV evaluation approach: "For performance evaluation, 5-fold cross-validation and independent test were used"
