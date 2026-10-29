---
name: contact-map-window-feature-extraction
description: Use when converting a binned contact map (e.g., 10 kb cooler with ICE
  'weight' column) into per-interaction feature vectors for machine-learning loop
  detection or similar pixel-level prediction tasks.
when_to_use_negative:
- The input is not a binned contact map (no .cool/.hic matrix) or lacks a valid normalization
  weight column, so window pixel values and relative ranks would be meaningless
- The goal is enrichment-based statistical loop calling (e.g., Fit-Hi-C q-values or
  HiCCUPS Poisson tests) rather than supervised feature-based classification
- The candidate interactions are only cross-chromosomal, since Peakachu scoring targets
  intra-chromosomal pixels within distance 6–300 bins
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Peakachu
  role: Implements the window feature-vector construction (11×11 window, relative
    ranks, P2LL) from a cooler/hic file and bedpe positives, via the 'peakachu train'
    subcommand
  repo: https://github.com/tariks/peakachu
- name: cooler
  role: Python library/environment dependency for reading the .cool contact matrix
    and its ICE weight column during feature extraction
- name: scikit-learn
  role: Consumes the extracted feature vectors to fit the random forest of 100 decision
    trees with 3-fold cross-validation grid search
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:4730f5f7097f3fdfeb90126486c56f764fa041d00a5425fa5096b610e7892eb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# contact-map-window-feature-extraction

## Summary

Extract local-window features from a genome-wide contact map (Hi-C .cool/.hic) to represent each candidate interaction as a fixed-size feature vector for supervised loop classification. Peakachu collects an 11×11 pixel window (width=5) around each interaction, encoding absolute normalized pixel values plus relative pixel ranks and the P2LL indicator.

## When to use

Use when converting a binned contact map (e.g., 10 kb cooler with ICE 'weight' column) into per-interaction feature vectors for machine-learning loop detection or similar pixel-level prediction tasks.

## When NOT to use

- The input is not a binned contact map (no .cool/.hic matrix) or lacks a valid normalization weight column, so window pixel values and relative ranks would be meaningless
- The goal is enrichment-based statistical loop calling (e.g., Fit-Hi-C q-values or HiCCUPS Poisson tests) rather than supervised feature-based classification
- The candidate interactions are only cross-chromosomal, since Peakachu scoring targets intra-chromosomal pixels within distance 6–300 bins

## Inputs

- A .cool file or .hic file of a genome-wide contact map at 10 kb resolution with a normalization weight column (e.g., ICE 'weight' or 'raw')
- A bedpe file defining the positive training interaction set (e.g., HiChIP/ChIA-PET-derived gm12878.mumbach.h3k27ac-hichip.hg19.bedpe)
- Resolution setting in bp (default 10000) and window width parameter (default width=5 giving 11×11 windows)

## Outputs

- Per-pixel/per-interaction 11×11 feature vectors of absolute pixel values, relative pixel ranks, and the P2LL indicator, labeled positive/negative
- Trained per-chromosome pickled random-forest models (23 models, one per chromosome) written to an output folder

## How to apply

Load the cooler file (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool at 10 kb resolution, using the 'weight' ICE normalization column) together with a bedpe positive training set (e.g., gm12878.mumbach.h3k27ac-hichip.hg19.bedpe). For each interaction, collect the 11×11 window of pixels centered on the interaction (width=5) and build feature vectors from the absolute pixel values, their relative ranks within the window, and the P2LL indicator (center-pixel ratio to the lower-left quadrant). Apply the minimum P2LL of 0.1 filter to retain positive-class samples. Split the dataset chromosome-wise (22 training chromosomes, 1 hold-out) so models never predict loops on the same map they were trained on, then pass these vectors to a random-forest fit (100 trees) with 3-fold cross-validation over splitting criterion, max depth, and class weights.

## Related tools

- **Peakachu** (Implements the window feature-vector construction (11×11 window, relative ranks, P2LL) from a cooler/hic file and bedpe positives, via the 'peakachu train' subcommand) — https://github.com/tariks/peakachu
- **cooler** (Python library/environment dependency for reading the .cool contact matrix and its ICE weight column during feature extraction)
- **scikit-learn** (Consumes the extracted feature vectors to fit the random forest of 100 decision trees with 3-fold cross-validation grid search)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- The trained model set contains exactly 23 pickled per-chromosome models (22 training chromosomes + 1 hold-out per model) in the output folder
- Positive-class samples passing extraction all satisfy the minimum P2LL indicator of 0.1
- Each feature vector has dimensionality consistent with an 11×11 window (width=5) including absolute values and relative ranks
- No hold-out chromosome's interactions were used to train that chromosome's model (leave-one-chromosome-out scheme respected)
- Downstream scoring on a hold-out chromosome (e.g., chr2, distance 6–300 bins, minimum-prob 0.5) produces usable per-pixel probabilities that pool into non-redundant bedpe loci with probability in column 7

## Limitations

- Feature extraction is calibrated for 10 kb-binned Hi-C matrices; a model can be trained with only ~200 manually annotated positive loops, but very sparse positive sets may underperform
- Relative ranks and P2LL depend on ICE balancing quality; raw signals ('raw' weight name) yield different features than normalized ones
- Extraction supports intra-chromosomal interactions within 6–300 bin distances for scoring; platform differences (Micro-C, DNA SPRITE) may require retrained models with adjusted probability cutoffs (0.97 CTCF, 0.92 H3K27ac for GM12878)

## Evidence

- [discussion] 11×11 window feature construction: "a corresponding 11 × 11 window centered at the interaction is collected from the Hi-C matrix. The ratio of the center pixel to the lower-left quadrant (P2LL) of this window is used as an indicator"
- [discussion] positive-class P2LL threshold: "the minimum P2LL for the positive class is set to 0.1"
- [readme] chromosome-wise split rationale: "The model for every chromosome was trained using interactions from all the other 22 chromosomes in the provided bedpe file. The purpose of this is to avoid Peakachu to predict loops from the same map"
- [methods] training command with weight column and bedpe: "peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe"
- [readme] width parameter defines window size: "Number of bins added to center of window. default width=5 corresponds to 11x11 windows"
