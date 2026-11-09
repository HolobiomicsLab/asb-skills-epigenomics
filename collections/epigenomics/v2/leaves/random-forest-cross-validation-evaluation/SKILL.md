---
name: random-forest-cross-validation-evaluation
description: Use when comparing feature subsets or classifiers for DNA 6mA sequence
  prediction on 41nt sequences and you need cross-validated AUC estimates (e.g. Binary
  vs Motif+Kmer+Binary) before freezing a feature matrix.
when_to_use_negative:
- Input is already a fixed feature table with no subset comparison needed — use direct
  model training (e.g. csDMA_training.py) instead.
- The goal is classifier-algorithm comparison rather than feature-subset comparison
  — classifier_selection.py covering RandomForest/GradientBoosting/AdaBoost/ExtraTrees/SVM
  is the appropriate entry point.
- Sequences are not 41nt cross-species 6mA samples (e.g. the M. musculus 1,934/1,934
  or rice 880/880 benchmarks) — feature counts and expected AUC values will not transfer.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: csDMA
  role: Provides feature_selection.py for encoding Motif, Kmer, Binary, and KSNPF
    feature subsets and the training scripts evaluated by 5-fold cross-validation
  repo: https://github.com/liuze-nwafu/csDMA
- name: scikit-learn
  role: Random Forest implementation (1,000 trees) with 5-fold cross-validation and
    ROC AUC scoring on the stratified training split
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:ae774d4c74a1e928feeeb247d6d9fb66e68937e64da17023cfc1dd76a21f12b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - Python
  - csDMA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# random-forest-cross-validation-evaluation

## Summary

Train a Random Forest classifier (1,000 trees in scikit-learn) on encoded DNA 6mA feature subsets and score each subset by AUC under 5-fold cross-validation on the 80% stratified training split of the cross-species csDMA dataset, to identify which Motif/Kmer/Binary/KSNPF combination to keep.

## When to use

Use when comparing feature subsets or classifiers for DNA 6mA sequence prediction on 41nt sequences and you need cross-validated AUC estimates (e.g. Binary vs Motif+Kmer+Binary) before freezing a feature matrix.

## When NOT to use

- Input is already a fixed feature table with no subset comparison needed — use direct model training (e.g. csDMA_training.py) instead.
- The goal is classifier-algorithm comparison rather than feature-subset comparison — classifier_selection.py covering RandomForest/GradientBoosting/AdaBoost/ExtraTrees/SVM is the appropriate entry point.
- Sequences are not 41nt cross-species 6mA samples (e.g. the M. musculus 1,934/1,934 or rice 880/880 benchmarks) — feature counts and expected AUC values will not transfer.

## Inputs

- Cross-species DNA 6mA training dataset (2,768 positive / 2,716 negative 41nt sequences, CD-HIT-EST threshold 0.80) from github.com/liuze-nwafu/csDMA
- Encoded feature matrices per subset: Binary-only, Motif+Binary, KSNPF-frequency+Binary, Motif+Kmer+Binary (336 k-mer features at k=2,3,4)
- 80% stratified training split labels

## Outputs

- cv_feature_subset_auc.csv with AUC per feature subset under 5-fold cross-validation
- Selected best feature-subset combination (Motif+Kmer+Binary, AUC ~0.871) used as the optimized feature matrix

## How to apply

Download the cross-species 6mA training dataset (2,768 positive / 2,716 negative 41nt samples) from the csDMA repository and take the 80% stratified training split. Encode each candidate feature subset (Binary-only, Motif+Binary, KSNPF-frequency+Binary, Motif+Kmer+Binary) using feature_selection.py, with k-mer features built at k=2, 3, and 4 (4^2 + 4^3 + 4^4 = 336 k-mer features). Train a Random Forest with 1,000 trees in scikit-learn per subset and evaluate with 5-fold cross-validation, computing AUC from the cross-validated predictions. Write the subset-vs-AUC results to cv_feature_subset_auc.csv and select the highest-AUC subset (expected: Motif+Kmer+Binary, AUC ~0.871) as the optimized feature matrix for downstream model training.

## Related tools

- **csDMA** (Provides feature_selection.py for encoding Motif, Kmer, Binary, and KSNPF feature subsets and the training scripts evaluated by 5-fold cross-validation) — https://github.com/liuze-nwafu/csDMA
- **scikit-learn** (Random Forest implementation (1,000 trees) with 5-fold cross-validation and ROC AUC scoring on the stratified training split)

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- Reproduced AUCs fall within tolerance of reported values: Binary ~0.866, Motif+Binary ~0.861, KSNPF+Binary ~0.862, Motif+Kmer+Binary ~0.871.
- Motif+Kmer+Binary emerges as the best subset across random fold splits.
- K-mer feature count equals 4^2 + 4^3 + 4^4 = 336 before concatenation with other subsets.
- cv_feature_subset_auc.csv contains exactly one AUC row per evaluated feature subset.
- Random Forest uses 1,000 trees and evaluation uses 5 folds on the 80% stratified training split only (no leakage from the 20% independent test set).

## Limitations

- AUC values are sensitive to random fold splits; exact reproduction of 0.866/0.861/0.862/0.871 is not guaranteed without fixing the random seed.
- csDMA performance on the cross-species dataset is relatively low compared with the M. musculus dataset, so absolute AUCs are moderate.
- csDMA was implemented in Python 2.7; running the RF/CV workflow under Python 3 may require adaptation.
- Genomic and structural feature encoding schemes were not yet incorporated, limiting the feature space evaluated.

## Evidence

- [intro] the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [intro] we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding.: "we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding."
- [intro] the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing: "the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing"
- [readme] 5-fold cross-validation was used to evaluate the performance of different classifier.: "5-fold cross-validation was used to evaluate the performance of different classifier."
- [methods] #This script implements the feature encoding scheme and can be auto involved in other scripts.: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
