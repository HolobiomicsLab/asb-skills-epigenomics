---
name: random-forest-cross-validated-hyperparameter-tuning
description: Use when training a supervised loop-detection model from a 10 kb Hi-C
  cooler file and a bedpe positive set (e.g., H3K27ac HiChIP), needing per-chromosome
  models and tuned hyperparameters before scoring pixels.
when_to_use_negative:
- You only need to score/pool pixels with pre-trained models — use peakachu score_chromosome/score_genome
  and pool instead of retraining
- The contact map is not binned at 10 kb or lacks a normalization weight column, so
  the 11x11-window features and ICE-weighted signals cannot be extracted as trained
- You have no orthogonal positive interaction set (ChIA-PET, HiChIP, Capture Hi-C)
  or at least ~200 annotated loops to define the training labels
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: Peakachu
  role: CLI framework implementing the per-chromosome random-forest training with
    leave-one-chromosome-out splitting ('peakachu train')
  repo: https://github.com/tariks/peakachu
- name: scikit-learn
  role: Provides the RandomForestClassifier and the 3-fold cross-validation grid search
    over splitting criterion, maximum tree depth, and class weights
- name: cooler
  role: Reads the 10 kb .cool contact map and its 'weight' (ICE) normalization column
    for feature extraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:2e0c30b3f7fd4b36e05c489dcbad6853fe39e4ec43a1ff1963ded16b96c33e0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# random-forest-cross-validated-hyperparameter-tuning

## Summary

Train per-chromosome random-forest classifiers with a 3-fold cross-validation grid search over splitting criterion, maximum tree depth, and class weights to score chromatin loop pixels, as implemented in Peakachu for genome-wide contact maps.

## When to use

Use when training a supervised loop-detection model from a 10 kb Hi-C cooler file and a bedpe positive set (e.g., H3K27ac HiChIP), needing per-chromosome models and tuned hyperparameters before scoring pixels.

## When NOT to use

- You only need to score/pool pixels with pre-trained models — use peakachu score_chromosome/score_genome and pool instead of retraining
- The contact map is not binned at 10 kb or lacks a normalization weight column, so the 11x11-window features and ICE-weighted signals cannot be extracted as trained
- You have no orthogonal positive interaction set (ChIA-PET, HiChIP, Capture Hi-C) or at least ~200 annotated loops to define the training labels

## Inputs

- A 10 kb-resolution .cool (or .hic) contact map, e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool with ICE weight column 'weight'
- A positive training set in bedpe format, e.g., gm12878.mumbach.h3k27ac-hichip.hg19.bedpe

## Outputs

- 23 pickled per-chromosome random-forest model files (one per chromosome, labeled by chromosome) in the models/ folder
- Trained classifiers ready for pixel scoring via peakachu score_chromosome/score_genome

## How to apply

Build 11x11-window feature vectors (width=5) from absolute pixel values, relative pixel ranks, and the P2LL indicator around each interaction in the 10 kb cooler file (ICE weight column 'weight'), keeping positive-class samples with minimum P2LL of 0.1. Split the data chromosome-wise — 22 training chromosomes and 1 hold-out chromosome per model — to avoid predicting loops from the same map used in training. Run a 3-fold cross-validation grid search over splitting criterion, maximum tree depth, and class weights (selecting via MCC), then fit a random forest of 100 decision trees with scikit-learn. Save 23 per-chromosome pickled models via 'peakachu train -r 10000 -O models'. Verify each model file exists in the models/ folder and that the hold-out chromosome scheme is respected before scoring.

## Related tools

- **Peakachu** (CLI framework implementing the per-chromosome random-forest training with leave-one-chromosome-out splitting ('peakachu train')) — https://github.com/tariks/peakachu
- **scikit-learn** (Provides the RandomForestClassifier and the 3-fold cross-validation grid search over splitting criterion, maximum tree depth, and class weights)
- **cooler** (Reads the 10 kb .cool contact map and its 'weight' (ICE) normalization column for feature extraction)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- 23 pickled model files are produced, one per chromosome, each trained on interactions from the other 22 chromosomes (leave-one-chromosome-out)
- Positive-class samples satisfy the minimum P2LL threshold of 0.1 before training
- Grid search uses 3-fold cross-validation and selects hyperparameters (splitting criterion, max depth, class weights) by MCC
- The fitted forest contains 100 decision trees, matching the paper's specification
- Downstream scoring of the hold-out chromosome (e.g., chr2) with the corresponding model file yields per-pixel probabilities that pool cleanly at threshold t=0.9

## Limitations

- Models are trained per chromosome to avoid predicting loops from the same map used in training; using a model on its own training chromosome risks overfitting
- A large proportion of predictions are shared with HiCCUPS/Fit-Hi-C, but each method detects a unique set of loops, so no single trained model captures the complete loop set
- As few as ~200 manually picked annotated loops suffice for training, but very sparse positive sets may still limit model generalizability across cell types

## Evidence

- [discussion] Chromosome-wise split rationale: "the whole dataset is split into a separate training and test dataset in a chromosome-wise manner, i.e., 22 chromosomes are used to train a model that is used to make final predictions in 1 hold-out"
- [discussion] Grid search specification: "we use a 3-fold cross-validation grid search to find the optimal hyperparameters (including splitting criterion, maximum tree depth and class weights)"
- [discussion] Positive-class P2LL filter: "the minimum P2LL for the positive class is set to 0.1"
- [methods] Training command: "peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe"
- [readme] README: purpose of per-chromosome models: "The model for every chromosome was trained using interactions from all the other 22 chromosomes in the provided bedpe file. The purpose of this is to avoid Peakachu to predict loops from the same map"
