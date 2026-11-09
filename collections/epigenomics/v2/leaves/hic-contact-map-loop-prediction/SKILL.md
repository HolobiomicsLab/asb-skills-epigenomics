---
name: hic-contact-map-loop-prediction
description: Use when you have a .cool/.hic genome-wide contact map at ~10 kb resolution
  plus a bedpe positive training set from an orthogonal assay (ChIA-PET, HiChIP, Capture
  Hi-C) and need supervised chromatin loop predictions.
when_to_use_negative:
- When no orthogonal positive interaction data (ChIA-PET, HiChIP, Capture Hi-C) is
  available to define the training set — although ~200 manually annotated loops were
  shown sufficient, no positive set means no supervised model
- When the goal is generic statistical enrichment calling rather than supervised classification
  — enrichment tools like HiCCUPS or Fit-Hi-C may be more appropriate
- When interactions are below the 6-bin lower distance bound or above the 300-bin
  upper bound (default score_chromosome distance limits)
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Peakachu
  role: Random forest framework providing train, score_chromosome, score_genome, and
    pool subcommands to train models and predict chromatin loops from contact maps
  repo: https://github.com/tariks/peakachu
- name: cooler
  role: Provides the .cool contact matrix format and ICE weight column (--clr-weight-name
    weight) consumed by peakachu train/score commands
- name: scikit-learn
  role: Backend used to fit the random forest of 100 decision trees per chromosome
    during training
- name: mamba
  role: Used to create the environment installing cooler, numba, scikit-learn=1.1.2,
    and joblib=1.1.0 before pip-installing peakachu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:fc961467f3c51cfca0dc9081b3dbb27b53ee7d03eae099a5d8578aa4f2ece4f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  - mamba
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hic-contact-map-loop-prediction

## Summary

Predict genome-wide chromatin loops from a Hi-C contact map by training Peakachu random forest models on an orthogonal positive interaction set (e.g. CTCF ChIA-PET or H3K27ac HiChIP bedpe), scoring every 10 kb pixel, and pooling probabilities into non-redundant loop calls.

## When to use

Use when you have a .cool/.hic genome-wide contact map at ~10 kb resolution plus a bedpe positive training set from an orthogonal assay (ChIA-PET, HiChIP, Capture Hi-C) and need supervised chromatin loop predictions.

## When NOT to use

- When no orthogonal positive interaction data (ChIA-PET, HiChIP, Capture Hi-C) is available to define the training set — although ~200 manually annotated loops were shown sufficient, no positive set means no supervised model
- When the goal is generic statistical enrichment calling rather than supervised classification — enrichment tools like HiCCUPS or Fit-Hi-C may be more appropriate
- When interactions are below the 6-bin lower distance bound or above the 300-bin upper bound (default score_chromosome distance limits)

## Inputs

- A .cool or .hic genome-wide contact map at 10 kb resolution (e.g. Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) with ICE balancing weights (weight column)
- A positive training interaction set in bedpe format (e.g. GM12878 CTCF ChIA-PET or gm12878.mumbach.h3k27ac-hichip.hg19.bedpe) from the Peakachu training-sets subfolder
- A trained model folder of pickled per-chromosome random forest models

## Outputs

- Per-pixel loop probability scores bedpe from score_genome
- Pooled, non-redundant loop calls bedpe (centroid loci; column 7 = predicted probability, final column = interaction frequency), e.g. ~13,360 genome-wide intrachromosomal loops for the GM12878 CTCF model
- Pickled random forest model files per chromosome (23 models)

## How to apply

Fit a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list defining positive samples; collect an 11x11 window centered on each interaction and compute the P2LL ratio (center pixel to lower-left quadrant, minimum P2LL 0.1 for positives). Train 23 per-chromosome random forest models (100 trees via scikit-learn) with 'peakachu train -r 10000 --clr-weight-name weight', where each chromosome's model is trained on the other 22 chromosomes to avoid predicting loops from the training map. Score every pixel genome-wide with 'peakachu score_genome' writing per-pixel probabilities to a scores bedpe, then run 'peakachu pool' with the model-specific probability cutoff (0.97 for the GM12878 CTCF model, 0.92 for H3K27ac) to select the most significant non-redundant loops. The pooled bedpe's 7th column holds predicted probability and the last column the interaction frequency; validate against orthogonal sources at the expected rate.

## Related tools

- **Peakachu** (Random forest framework providing train, score_chromosome, score_genome, and pool subcommands to train models and predict chromatin loops from contact maps) — https://github.com/tariks/peakachu
- **cooler** (Provides the .cool contact matrix format and ICE weight column (--clr-weight-name weight) consumed by peakachu train/score commands)
- **scikit-learn** (Backend used to fit the random forest of 100 decision trees per chromosome during training)
- **mamba** (Used to create the environment installing cooler, numba, scikit-learn=1.1.2, and joblib=1.1.0 before pip-installing peakachu)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- The GM12878 CTCF ChIA-PET-trained model applied to the GM12878 Hi-C matrix yields ~13,360 genome-wide intrachromosomal loops at probability threshold 0.97
- Approximately 83% (11,143/13,360) of predicted loops are supported by at least one orthogonal validation source
- Training produces 23 pickled models, one per chromosome, each trained on interactions from the other 22 chromosomes
- Output bedpe schema check: 7th column contains predicted probability, final column contains interaction frequency
- Only pixels with probability above --minimum-prob (default 0.5; 0.97 cutoff applied via pool for the CTCF model) are output

## Limitations

- Probability cutoffs for cell lines other than GM12878 (0.97 CTCF, 0.92 H3K27ac) are only in Supplementary Data 2
- No software version numbers stated in the paper for scikit-learn at training time (README pins scikit-learn=1.1.2, joblib=1.1.0)
- Peakachu, HiCCUPS, and Fit-Hi-C each detect a unique set of loops, so the complete loop set may not be captured by any single method
- Training requires a positive interaction list from an orthogonal assay; other platforms like DNA SPRITE historically lacked dedicated loop-detection algorithms

## Evidence

- [readme] Train per-chromosome RF with chromosome-wise split rationale: "This will train 23 random forest models, each labeled by a chromosome. The model for every chromosome was trained using interactions from all the other 22 chromosomes"
- [discussion] P2LL feature and window: "a corresponding 11 × 11 window centered at the interaction is collected from the Hi-C matrix. The ratio of the center pixel to the lower-left quadrant (P2LL)"
- [methods] Pool function purpose: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [discussion] Probability cutoffs for CTCF and H3K27ac models: "we set the cutoff to 0.97 and 0.92 for the CTCF model and the H3K27ac model, respectively."
- [discussion] Training components requirement: "Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set."
- [discussion] Random forest size: "Using scikit-learn, the training set is then used as input to fit a random forest of 100 decision trees"
