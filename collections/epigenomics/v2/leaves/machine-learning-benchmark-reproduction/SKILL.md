---
name: machine-learning-benchmark-reproduction
description: Use when a paper reports classifier or feature-subset performance metrics
  (e.g. AUC values per feature combination) and the goal is to re-run the released
  code/data to verify those numbers.
when_to_use_negative:
- The input is already an encoded numeric feature matrix — this skill starts from
  raw 41nt sequence datasets and builds encodings via feature_selection.py.
- The goal is independent-test evaluation or model deployment (use csDMA_training.py
  / csDMA_indepedent_testing.py instead of feature-subset CV comparison).
- The target dataset is a different species benchmark (e.g. the M. musculus 1,934/1,934
  or rice 880/880 datasets) whose reported numbers may not match the cross-species
  values.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: csDMA
  role: Provides the feature_selection.py encoding scheme and classifier-selection
    scripts used to reproduce the feature-subset AUC comparison
  repo: https://github.com/liuze-nwafu/csDMA
- name: scikit-learn
  role: Trains the Random Forest (1,000 trees) and runs the 5-fold cross-validation
    used to compute per-subset AUC
- name: CD-HIT-EST
  role: Was applied at threshold 0.80 to build the non-redundant cross-species dataset
    that this reproduction starts from
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:ef1fee444d59659d378a4d7dc24af076b3bfa6b919e11faf86839f8ebdaa00ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - CD-HIT-EST
  - Python
  - csDMA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# machine-learning-benchmark-reproduction

## Summary

Reproduces a published machine-learning benchmark from a paper's own scripts and datasets — here, re-deriving the csDMA feature-subset comparison (Binary, Motif+Binary, KSNPF+Binary, Motif+Kmer+Binary) with 5-fold cross-validated Random Forest AUC on the cross-species 6mA dataset.

## When to use

Use when a paper reports classifier or feature-subset performance metrics (e.g. AUC values per feature combination) and the goal is to re-run the released code/data to verify those numbers.

## When NOT to use

- The input is already an encoded numeric feature matrix — this skill starts from raw 41nt sequence datasets and builds encodings via feature_selection.py.
- The goal is independent-test evaluation or model deployment (use csDMA_training.py / csDMA_indepedent_testing.py instead of feature-subset CV comparison).
- The target dataset is a different species benchmark (e.g. the M. musculus 1,934/1,934 or rice 880/880 datasets) whose reported numbers may not match the cross-species values.

## Inputs

- Cross-species DNA 6mA training dataset (2,768 positive / 2,716 negative 41nt FASTA samples) from github.com/liuze-nwafu/csDMA
- feature_selection.py (feature encoding script from the csDMA repo)
- classifier_selection.py (five-classifier 5-fold CV script from the csDMA repo)
- Motif, Kmer, KSNPF-frequency, and Binary feature-subset matrices

## Outputs

- cv_feature_subset_auc.csv with cross-validated AUC per feature subset
- Reproduced AUC values for Binary (~0.866), Motif+Binary (~0.861), KSNPF+Binary (~0.862), Motif+Kmer+Binary (~0.871)
- Identified best feature-subset combination (Motif+Kmer+Binary)

## How to apply

Download the cross-species DNA 6mA training dataset (2,768 positive / 2,716 negative 41nt samples, CD-HIT-EST threshold 0.80) from the csDMA GitHub repository and the 80% stratified training split. Encode sequences using feature_selection.py, building the compared feature subsets: Binary-only, Motif+Binary, KSNPF-frequency+Binary, and Motif+Kmer+Binary, with k-mer features at k=2, 3, and 4 (4^2 + 4^3 + 4^4 = 336 k-mer features). Train a Random Forest with 1,000 trees in scikit-learn per subset and evaluate each by 5-fold cross-validation, computing AUC from cross-validated predictions; write results to cv_feature_subset_auc.csv. Judge success by matching the reported values within tolerance for random fold splits: Binary ~0.866, Motif+Binary ~0.861, KSNPF+Binary ~0.862, Motif+Kmer+Binary ~0.871, with Motif+Kmer+Binary emerging as the best combination.

## Related tools

- **csDMA** (Provides the feature_selection.py encoding scheme and classifier-selection scripts used to reproduce the feature-subset AUC comparison) — https://github.com/liuze-nwafu/csDMA
- **scikit-learn** (Trains the Random Forest (1,000 trees) and runs the 5-fold cross-validation used to compute per-subset AUC)
- **CD-HIT-EST** (Was applied at threshold 0.80 to build the non-redundant cross-species dataset that this reproduction starts from)

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- Binary-only subset yields AUC ≈ 0.866 by 5-fold cross-validation
- Motif+Binary and KSNPF+Binary subsets yield AUC ≈ 0.861 and ≈ 0.862 respectively
- Motif+Kmer+Binary is the best subset with AUC ≈ 0.871, matching the published value within tolerance for random fold splits
- k-mer feature count equals 4^2 + 4^3 + 4^4 = 336 features
- All AUC values are computed on the 80% stratified training split with 5-fold cross-validation, not the independent 20% test set

## Limitations

- AUC values will not match exactly because fold splits are random; only approximate agreement within tolerance is expected.
- csDMA was implemented in Python 2.7, so reproduction may require a Python 2.7 environment for the original scripts.
- csDMA performance is relatively low on the rice and cross-species datasets, and genomic/structural feature encodings are not incorporated.

## Evidence

- [intro] the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [intro] we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding.: "we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding."
- [intro] the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing: "the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing"
- [readme] 5-fold cross-validation was used to evaluate the performance of different classifier.: "5-fold cross-validation was used to evaluate the performance of different classifier."
- [readme] #This script implements the feature encoding scheme and can be auto involved in other scripts.: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
