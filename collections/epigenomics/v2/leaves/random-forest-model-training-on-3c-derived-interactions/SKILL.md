---
name: random-forest-model-training-on-3c-derived-interactions
description: Use when you have a 10 kb-binned .cool or .hic Hi-C matrix and a bedpe
  list of positive interactions (ChIA-PET, HiChIP, Capture Hi-C) and need chromosome-wise
  random forest models for loop prediction.
when_to_use_negative:
- You only need per-pixel loop probabilities or pooled loop calls and already possess
  trained models — use peakachu score_genome plus pool instead of retraining.
- The input contact matrix is raw without a balancing weight column and you cannot
  compute ICE weights — specify '--clr-weight-name raw' consciously or fix balancing
  first, since the workflow assumes 'weight' normalization.
- The positive interaction set is not from an orthogonal genome-wide assay mapped
  to the same genome build/resolution as the contact map (e.g., a small hand-curated
  list unrelated to the organism or assembly).
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0622
tools:
- name: Peakachu
  role: Command-line framework providing the 'train' subcommand that fits per-chromosome
    random forest models from a 10 kb cooler and a positive bedpe
  repo: https://github.com/tariks/peakachu
- name: scikit-learn
  role: Random forest implementation (100 decision trees) and 3-fold cross-validation
    grid search used during training
- name: cooler
  role: Reads the .cool contact matrix and its ICE 'weight' column for balanced contact
    signals during feature extraction
- name: mamba
  role: Creates the pinned environment (cooler, numba, scikit-learn=1.1.2, joblib=1.1.0)
    before pip-installing peakachu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:5700785ad93adb448cec0cbff1c832dc26f0299c21e18ae3f7b8f5577ac8cfe3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  - mamba
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# random-forest-model-training-on-3c-derived-interactions

## Summary

Train per-chromosome Peakachu random forest models that distinguish chromatin loops from non-loop contacts in a Hi-C contact map, using orthogonal 3C-derived interactions (e.g., CTCF ChIA-PET or H3K27ac HiChIP bedpe files) as the positive training set. This is the supervised-learning core of Peakachu's loop-detection framework.

## When to use

Use when you have a 10 kb-binned .cool or .hic Hi-C matrix and a bedpe list of positive interactions (ChIA-PET, HiChIP, Capture Hi-C) and need chromosome-wise random forest models for loop prediction.

## When NOT to use

- You only need per-pixel loop probabilities or pooled loop calls and already possess trained models — use peakachu score_genome plus pool instead of retraining.
- The input contact matrix is raw without a balancing weight column and you cannot compute ICE weights — specify '--clr-weight-name raw' consciously or fix balancing first, since the workflow assumes 'weight' normalization.
- The positive interaction set is not from an orthogonal genome-wide assay mapped to the same genome build/resolution as the contact map (e.g., a small hand-curated list unrelated to the organism or assembly).

## Inputs

- Hi-C contact matrix as a .cool file at 10 kb resolution (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) or a .hic file (supported by Peakachu >=1.1.2)
- Positive training set of 3C-derived interactions in bedpe format (e.g., GM12878 CTCF ChIA-PET interactions, or gm12878.mumbach.h3k27ac-hichip.hg19.bedpe) from the Peakachu training-sets subfolder
- ICE balancing weight column in the cooler ('--clr-weight-name weight')

## Outputs

- Folder of 23 pickled per-chromosome random forest models (scikit-learn, 100 trees), each trained on interactions from the other 22 chromosomes
- Trained models suitable for downstream 'peakachu score_genome'/'score_chromosome' per-pixel loop probability scoring

## How to apply

Download the Hi-C cooler (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) and the positive interaction bedpe from the Peakachu training-sets subfolder, then create a mamba environment with cooler, numba, scikit-learn=1.1.2, and joblib=1.1.0 and install peakachu from PyPI. Run 'peakachu train -r 10000' with '--clr-weight-name weight' so ICE-normalized weights are used, passing the bedpe via -b and an output folder via -O; this trains 23 random forest models (100 decision trees each), where the model for every chromosome is trained on interactions from all other 22 chromosomes to avoid predicting loops from the map used in training (chromosome-wise split, preventing overfitting). During feature extraction, an 11x11 window centered at each interaction is collected from the matrix (width=5) and the ratio of the center pixel to the lower-left quadrant (P2LL) is used as the indicator, with a minimum P2LL of 0.1 for positive-class samples. Hyperparameters (splitting criterion, maximum tree depth, class weights) are selected via a 3-fold cross-validation grid search with MCC as the selection metric. The resulting pickled models in the output folder are subsequently consumed by score_chromosome/score_genome; the same procedure applies whether the bedpe derives from CTCF ChIA-PET (GM12878, 92,807 interactions) or H3K27ac HiChIP.

## Related tools

- **Peakachu** (Command-line framework providing the 'train' subcommand that fits per-chromosome random forest models from a 10 kb cooler and a positive bedpe) — https://github.com/tariks/peakachu
- **scikit-learn** (Random forest implementation (100 decision trees) and 3-fold cross-validation grid search used during training)
- **cooler** (Reads the .cool contact matrix and its ICE 'weight' column for balanced contact signals during feature extraction)
- **mamba** (Creates the pinned environment (cooler, numba, scikit-learn=1.1.2, joblib=1.1.0) before pip-installing peakachu)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- The output folder contains exactly 23 pickled model files, one per chromosome, as stated in the README ('This will train 23 random forest models, each labeled by a chromosome').
- Downstream scoring with the trained CTCF ChIA-PET model on GM12878 Hi-C reproduces ~13,360 genome-wide intrachromosomal loops at probability cutoff 0.97, with ~83% (11,143) supported by at least one orthogonal source.
- Training log/usage confirms 'peakachu train -r 10000 --clr-weight-name weight -O models -b <bedpe>' was used (ICE weights, not raw), matching the methods workflow.
- Positive-class samples satisfy the minimum P2LL of 0.1 filter, and the chromosome-wise split guarantees the scored chromosome was never seen in its own training set.
- Hyperparameter grid search reports MCC as the selection metric across splitting criterion, maximum tree depth, and class weights.

## Limitations

- Only ~200 manually picked annotated loops were sufficient to train a model, but performance depends on the quality and genome-build compatibility of the positive set.
- Training on interactions from a map and then scoring the same map is deliberately avoided via the chromosome-wise split; violating this risks overfitting.
- Peakachu detects a unique set of short-range interactions; the complete set of chromatin loops might not be captured by a single method (Peakachu, HiCCUPS, and Fit-Hi-C each detect a unique set).
- Probability cutoffs differ per model and cell line (0.97 for CTCF, 0.92 for H3K27ac in GM12878); cutoffs for other cell lines are only available in Supplementary Data 2.
- No software version numbers are stated in the article for some dependencies beyond the README's scikit-learn=1.1.2 and joblib=1.1.0 pins.

## Evidence

- [readme] chromosome-wise split rationale: "The model for every chromosome was trained using interactions from all the other 22 chromosomes in the provided bedpe file."
- [discussion] feature window and P2LL: "a corresponding 11 × 11 window centered at the interaction is collected from the Hi-C matrix. The ratio of the center pixel to the lower-left quadrant (P2LL)"
- [discussion] grid search metric: "we use a 3-fold cross-validation grid search to find the optimal hyperparameters (including splitting criterion, maximum tree depth and class weights)"
- [methods] train command with weights: "peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe"
- [discussion] P2LL threshold: "the minimum P2LL for the positive class is set to 0.1"
- [discussion] minimal positive set size: "we manually picked ~200 annotated loops (Supplementary Data 3) and found it was enough for Peakachu to train a model to perform genome-wide predictions"
