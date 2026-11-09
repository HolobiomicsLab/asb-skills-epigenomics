---
name: random-forest-model-training-hic
description: Use when a 10 kb-binned contact map (.cool or .hic) and a positive interaction
  list (bedpe from ChIA-PET, HiChIP, or annotated loops) are available and supervised
  chromatin loop detection is needed.
when_to_use_negative:
- The input is a contact map from a platform with no binned 10 kb matrix or no orthogonal
  positive interaction set available for training
- The goal is de novo statistical peak calling rather than supervised loop classification
  — enrichment-based tools (HiCCUPS, Fit-Hi-C, MAPS, CHiCAGO) are the comparison baselines
  here, not the method being trained
- The contact map resolution differs substantially from 10 kb and rescaling/rebinning
  cannot be performed, since models are fit at 10 kb binning (default -r 10000)
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: Peakachu
  role: Random Forest classification framework; 'peakachu train' fits per-chromosome
    models from the 10 kb contact matrix and bedpe positive set
  repo: https://github.com/tariks/peakachu
- name: scikit-learn
  role: Underlying implementation of the random forest (100 decision trees) and 3-fold
    cross-validation grid search
- name: cooler
  role: Reads the .cool contact matrix and its weight (ICE) or raw balancing columns
    for signal normalization
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:1a1c18c685ad01069e9256edfcfd8702113d46c266079e8be2c35b0b62888511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# random-forest-model-training-hic

## Summary

Train chromosome-wise Peakachu Random Forest classifiers that distinguish chromatin loops from non-loops in 10 kb-binned genome-wide contact maps (.cool/.hic), using positive interaction sets (e.g. CTCF ChIA-PET or H3K27ac HiChIP bedpe) and negative randomly sampled contacts, to enable platform-agnostic loop prediction across Hi-C, Micro-C, and DNA SPRITE.

## When to use

Use when a 10 kb-binned contact map (.cool or .hic) and a positive interaction list (bedpe from ChIA-PET, HiChIP, or annotated loops) are available and supervised chromatin loop detection is needed.

## When NOT to use

- The input is a contact map from a platform with no binned 10 kb matrix or no orthogonal positive interaction set available for training
- The goal is de novo statistical peak calling rather than supervised loop classification — enrichment-based tools (HiCCUPS, Fit-Hi-C, MAPS, CHiCAGO) are the comparison baselines here, not the method being trained
- The contact map resolution differs substantially from 10 kb and rescaling/rebinning cannot be performed, since models are fit at 10 kb binning (default -r 10000)

## Inputs

- 10 kb-binned contact matrix as .cool or .hic file (e.g. Rao2014-GM12878-MboI-allreps-filtered.10kb.cool)
- Positive training set in bedpe format (e.g. gm12878.mumbach.h3k27ac-hichip.hg19.bedpe from the peakachu training-sets subfolder)
- clr weight column name in the cooler ('weight' for ICE normalization or 'raw')

## Outputs

- 23 pickled per-chromosome random forest model files in the -O output folder
- Trained models ready for downstream 'peakachu score_genome'/'score_chromosome' loop prediction

## How to apply

Fit a Peakachu model from two components: a Hi-C matrix binned to 10 kb and a bedpe interaction list defining the positive training set. For each positive interaction, collect an 11x11 window (width=5) from the contact matrix and compute the P2LL indicator (center-pixel ratio to the lower-left quadrant); positive-class samples require minimum P2LL of 0.1. Split data chromosome-wise — 22 chromosomes train the model applied to 1 hold-out chromosome — to avoid predicting loops from the same map used for training without overfitting, and use a 3-fold cross-validation grid search (splitting criterion, maximum tree depth, class weights) selecting by MCC to tune hyperparameters of a 100-tree scikit-learn random forest. Run 'peakachu train -r 10000' with the appropriate --clr-weight-name: 'weight' for ICE-normalized Hi-C, or 'raw' for raw-signal platforms such as Micro-C and DNA SPRITE. Several hundred positive points suffice — ~200 manually picked annotated loops were enough for genome-wide prediction.

## Related tools

- **Peakachu** (Random Forest classification framework; 'peakachu train' fits per-chromosome models from the 10 kb contact matrix and bedpe positive set) — https://github.com/tariks/peakachu
- **scikit-learn** (Underlying implementation of the random forest (100 decision trees) and 3-fold cross-validation grid search)
- **cooler** (Reads the .cool contact matrix and its weight (ICE) or raw balancing columns for signal normalization)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- The output folder contains 23 model files, each labeled by a chromosome, confirming chromosome-wise splitting (22 train / 1 hold-out)
- Positive-class training samples satisfy the minimum P2LL >= 0.1 filter
- 3-fold cross-validation grid search selects hyperparameters (splitting criterion, max tree depth, class weights) by maximizing MCC
- Models trained on ICE-normalized Hi-C use --clr-weight-name weight, while Micro-C and DNA SPRITE models use --clr-weight-name raw
- Models generalize: applied to Micro-C H1-ESC they yielded 16,298 loops (74% Hi-C overlap) and to sparse GM12878 DNA SPRITE 1,731 loops (77% Hi-C-validated)

## Limitations

- A model trained only on several hundred positives (~200 manually picked annotated loops) is possible, but performance depends on the quality and platform-concordance of the positive bedpe set
- Chromosome-wise splitting means each chromosome's predictions come from a model trained on other chromosomes; the same map must not be used for both training and prediction to avoid overfitting
- Platform-specific normalization matters: raw values (--clr-weight-name raw) are required for Micro-C and DNA SPRITE, whereas ICE weights ('weight') apply to Hi-C; probability cutoffs differ per model (0.97 CTCF, 0.92 H3K27ac in GM12878) and for other cell lines are only in Supplementary Data 2

## Evidence

- [discussion] Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set.: "Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set."
- [readme] This will train 23 random forest models, each labeled by a chromosome. The model for every chromosome was trained using interactions from all the other 22 chromosomes in the provided bedpe file.: "The model for every chromosome
was trained using interactions from all the other 22 chromosomes in the provided bedpe file."
- [discussion] the minimum P2LL for the positive class is set to 0.1: "the minimum P2LL for the positive class is set to 0.1"
- [discussion] we use a 3-fold cross-validation grid search to find the optimal hyperparameters (including splitting criterion, maximum tree depth and class weights): "we use a 3-fold cross-validation grid search to find the optimal hyperparameters (including splitting criterion, maximum tree depth and class weights)"
- [discussion] we manually picked ~200 annotated loops (Supplementary Data 3) and found it was enough for Peakachu to train a model to perform genome-wide predictions: "we manually picked ~200 annotated loops (Supplementary Data 3) and found it was enough for Peakachu to train a model to perform genome-wide predictions"
