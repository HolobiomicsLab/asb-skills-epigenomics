---
name: random-forest-model-training-per-chromosome
description: Use when you have a .cool or .hic genome-wide contact map and an orthogonal
  positive interaction set (e.g. HiChIP/ChIA-PET bedpe) and need supervised loop classifiers
  for genome-wide prediction.
when_to_use_negative:
- You have no orthogonal positive interaction set (HiChIP, ChIA-PET, or manually annotated
  loops) — training requires a positive training list in bedpe format
- The contact map is not binned (resolution is unknown or far from 10 kb) — the framework
  requires a Hi-C matrix binned to 10 kb
- You only need generic statistical peak enrichment rather than supervised loop classification
  — enrichment-based tools like Fit-Hi-C or HiCCUPS do not require training data
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3474
tools:
- name: Peakachu
  role: Supervised random forest framework whose 'train' subcommand fits per-chromosome
    models from the cooler and bedpe inputs
  repo: https://github.com/tariks/peakachu
- name: scikit-learn
  role: Provides the random forest (100 decision trees) and 3-fold cross-validation
    grid search implementation used during training
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:52a62475663583b75a8cf048b57fa8130de73e073b344ff324bb101f2ac5f4b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# random-forest-model-training-per-chromosome

## Summary

Train chromosome-labeled random forest classifiers (Peakachu) that distinguish chromatin loops from non-loop pixels using a 10 kb-binned Hi-C contact matrix and a positive interaction list in bedpe format, with a chromosome-wise train/hold-out split to avoid overfitting.

## When to use

Use when you have a .cool or .hic genome-wide contact map and an orthogonal positive interaction set (e.g. HiChIP/ChIA-PET bedpe) and need supervised loop classifiers for genome-wide prediction.

## When NOT to use

- You have no orthogonal positive interaction set (HiChIP, ChIA-PET, or manually annotated loops) — training requires a positive training list in bedpe format
- The contact map is not binned (resolution is unknown or far from 10 kb) — the framework requires a Hi-C matrix binned to 10 kb
- You only need generic statistical peak enrichment rather than supervised loop classification — enrichment-based tools like Fit-Hi-C or HiCCUPS do not require training data

## Inputs

- 10 kb-resolution cooler file (e.g. Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) with a 'weight' balancing column
- Positive training set in bedpe format (e.g. gm12878.mumbach.h3k27ac-hichip.hg19.bedpe); ~200 manually annotated loops can suffice

## Outputs

- Folder of 23 pickled random forest models (100 trees each), labeled by chromosome, for use with peakachu score_genome/score_chromosome

## How to apply

Provide a 10 kb-binned cooler (e.g. Rao2014-GM12878-MboI-allreps-filtered.10kb.cool with the 'weight' balancing column) and a positive training bedpe such as gm12878.mumbach.h3k27ac-hichip.hg19.bedpe to 'peakachu train -r 10000'. For each interaction, Peakachu collects an 11x11 window centered on the pixel (width=5) and uses the P2LL indicator (center pixel over lower-left quadrant ratio); positives require a minimum P2LL of 0.1. It fits 23 random forest models of 100 trees each, one per chromosome, where each chromosome's model is trained on interactions from the other 22 chromosomes (22-train/1-hold-out) so predictions are never made on the training map. Hyperparameters (splitting criterion, maximum tree depth, class weights) are selected by a 3-fold cross-validation grid search using MCC as the selection metric. The output is a folder of pickled per-chromosome models consumed downstream by 'peakachu score_genome' or 'score_chromosome'.

## Related tools

- **Peakachu** (Supervised random forest framework whose 'train' subcommand fits per-chromosome models from the cooler and bedpe inputs) — https://github.com/tariks/peakachu
- **scikit-learn** (Provides the random forest (100 decision trees) and 3-fold cross-validation grid search implementation used during training)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- The training command reports 23 chromosome-labeled models written to the -O output folder (22 train / 1 hold-out per model)
- Positive-class samples pass the minimum P2LL of 0.1 filter before model fitting
- A model trained on ~200 manually picked annotated loops should be sufficient to support genome-wide predictions (per the paper's discussion)
- Downstream score_genome outputs with a depth-matched model should yield loop calls concordant with the full-depth predictions (e.g. ~85% concordance at 1.5% down-sampling)

## Limitations

- No explicit software version numbers are stated for scikit-learn beyond the README's pinned scikit-learn=1.1.2 and joblib=1.1.0
- A model trained on one map's positives should not predict loops on that same map; the chromosome-wise split exists precisely to avoid predicting from the training map
- Probability cutoffs (0.92 H3K27ac, 0.97 CTCF) are cell-line-specific for GM12878; cutoffs for other cell lines are only in Supplementary Data 2

## Evidence

- [discussion] Chromosome-wise split rationale: "the whole dataset is split into a separate training and test dataset in a chromosome-wise manner, i.e., 22 chromosomes are used to train a model that is used to make final predictions in 1 hold-out"
- [discussion] Training inputs and window/P2LL features: "Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set."
- [discussion] Cross-validation hyperparameter search: "we use a 3-fold cross-validation grid search to find the optimal hyperparameters (including splitting criterion, maximum tree depth and class weights)"
- [discussion] Positive-class P2LL filter: "the minimum P2LL for the positive class is set to 0.1"
- [readme] README train invocation: "This will train 23 random forest models, each labeled by a chromosome. The model for every chromosome was trained using interactions from all the other 22 chromosomes"
