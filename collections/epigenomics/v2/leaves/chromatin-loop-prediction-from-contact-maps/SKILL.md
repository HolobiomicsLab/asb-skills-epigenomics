---
name: chromatin-loop-prediction-from-contact-maps
description: Use when you have a 10 kb-binned .cool/.hic contact map and a positive
  interaction set (ChIA-PET/HiChIP/Capture Hi-C bedpe) and need supervised, data-driven
  chromatin loop calls that are robust to low sequencing depth.
when_to_use_negative:
- The goal is an unsupervised/enrichment-based call (local Poisson background as in
  HiCCUPS or distance-dependent splines as in Fit-Hi-C) rather than supervised classification
  against orthogonal positives
- No positive interaction training set (ChIA-PET/HiChIP/Capture Hi-C bedpe) or suitable
  pre-trained model is available for the cell type/platform
- The contact map is not binned near 10 kb resolution or lacks an ICE weight column
  (and raw signals are not appropriate for the chosen model)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: Peakachu
  role: 'Random Forest classification framework: train per-chromosome models, score
    pixels genome-wide, and pool non-redundant loop calls'
  repo: https://github.com/tariks/peakachu
- name: cooler
  role: Loads and normalizes .cool/.mcool contact maps at 10 kb resolution with the
    ICE 'weight' column
- name: scikit-learn
  role: Fits the Random Forest of 100 decision trees with 3-fold cross-validated grid
    search over splitting criterion, tree depth, and class weights
- name: HiCCUPS
  role: Enrichment-based benchmark loop caller (Juicer tool) used for overlap comparison
    with Peakachu predictions
  repo: https://github.com/aidenlab/juicer
- name: Fit-Hi-C
  role: Distance-dependent spline benchmark loop caller (q-value < 1e-5 filter, top
    140,000 pooled) used for method comparison
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  - task_003
  - task_007
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:fdde0deafdb967a443c149d5d29475d39eda635132b07406ccf29c089f5b161b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Fit-Hi-C
  - HiCCUPS
  - Peakachu
  - cooler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# chromatin-loop-prediction-from-contact-maps

## Summary

Predict chromatin loops from genome-wide contact maps (Hi-C, Micro-C, DNA SPRITE) using Peakachu, a Random Forest classifier trained on orthogonal positive interaction sets (e.g., CTCF ChIA-PET or H3K27ac HiChIP bedpe files) rather than statistical enrichment of pixels.

## When to use

Use when you have a 10 kb-binned .cool/.hic contact map and a positive interaction set (ChIA-PET/HiChIP/Capture Hi-C bedpe) and need supervised, data-driven chromatin loop calls that are robust to low sequencing depth.

## When NOT to use

- The goal is an unsupervised/enrichment-based call (local Poisson background as in HiCCUPS or distance-dependent splines as in Fit-Hi-C) rather than supervised classification against orthogonal positives
- No positive interaction training set (ChIA-PET/HiChIP/Capture Hi-C bedpe) or suitable pre-trained model is available for the cell type/platform
- The contact map is not binned near 10 kb resolution or lacks an ICE weight column (and raw signals are not appropriate for the chosen model)

## Inputs

- 10 kb-binned .cool/.mcool contact map (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool), with ICE 'weight' column or raw values
- Positive training set in bedpe format (e.g., gm12878.mumbach.h3k27ac-hichip.hg19.bedpe from the peakachu training-sets subfolder)
- Pickled per-chromosome Random Forest models (*.pkl), e.g., CTCF-ChIAPET-peakachu-pretrained.*.10kb.pkl

## Outputs

- Per-chromosome or genome-wide per-pixel probability bedpe from score_genome/score_chromosome (probability in column 7, interaction frequency in final column)
- Pooled non-redundant loop bedpe of centroid loci above the probability threshold (e.g., GM12878-chr2-loops.bedpe)

## How to apply

Fit 23 chromosome-labeled Random Forest models (100 decision trees, scikit-learn) with 'peakachu train -r 10000', supplying a 10 kb cooler (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) and a positive bedpe such as gm12878.mumbach.h3k27ac-hichip.hg19.bedpe; each chromosome's model is trained on interactions from the other 22 chromosomes (leave-one-chromosome-out) to avoid overfitting. Feature vectors are built from 11×11 windows (width=5) of absolute pixel values plus relative pixel ranks, with the center-pixel to lower-left-quadrant ratio (P2LL, minimum 0.1 for positives) as an indicator. Score all intra-chromosomal pixels (distance 6–300 bins) with 'peakachu score_genome'/'score_chromosome', setting --clr-weight-name to 'weight' for ICE-normalized Hi-C or 'raw' for raw platforms (CTCF ChIA-PET-trained scoring of Hi-C, Micro-C, DNA SPRITE, and RAD21-AID auxin maps). Pool per-pixel probabilities with 'peakachu pool' using model-specific cutoffs (0.97 for the CTCF model, 0.92 for the H3K27ac model; default minimum-prob 0.5, typically raised to 0.9–0.95) to select non-redundant centroid loci via 1D anchor-peak definition and DBSCAN clustering. Check sequencing depth first with 'peakachu depth' to select a depth-matched pre-trained model when not training de novo.

## Related tools

- **Peakachu** (Random Forest classification framework: train per-chromosome models, score pixels genome-wide, and pool non-redundant loop calls) — https://github.com/tariks/peakachu
- **cooler** (Loads and normalizes .cool/.mcool contact maps at 10 kb resolution with the ICE 'weight' column)
- **scikit-learn** (Fits the Random Forest of 100 decision trees with 3-fold cross-validated grid search over splitting criterion, tree depth, and class weights)
- **HiCCUPS** (Enrichment-based benchmark loop caller (Juicer tool) used for overlap comparison with Peakachu predictions) — https://github.com/aidenlab/juicer
- **Fit-Hi-C** (Distance-dependent spline benchmark loop caller (q-value < 1e-5 filter, top 140,000 pooled) used for method comparison)

## Examples

```
peakachu train -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O models -b gm12878.mumbach.h3k27ac-hichip.hg19.bedpe
```

## Evaluation signals

- CTCF ChIA-PET-trained model applied to RAD21-AID auxin-depleted HCT-116 Hi-C should yield ~19 loops vs 11,814 in untreated at matched depth (~0.2% FDR)
- H3K27ac HiChIP-trained model on GM12878 Hi-C at cutoff 0.92 should reproduce ~13,292 loops with ~65% overlap with CTCF-model predictions and ~84% single-source validation
- Sequencing-depth robustness: 1.5% down-sampled GM12878 (~30M reads) should give ~2363 loops with ~90% orthogonal validation, and 85–88% concordance with full-depth predictions at 50%/10%/1.5%
- Cross-platform transfer: Micro-C H1-ESC should yield 16,298 loops (74% Hi-C overlap) and sparse GM12878 DNA SPRITE 1731 loops (77% Hi-C validation)
- Pooled bedpe must carry the predicted probability in column 7 and the interaction frequency in the final column, with all retained calls above the pooling threshold

## Limitations

- The complete set of chromatin loops might not be captured by a single method: Peakachu, HiCCUPS, and Fit-Hi-C each detect a unique set of loops with different distance distributions, CTCF binding, and validation ratios
- Platforms such as DNA SPRITE lack dedicated loop-detection algorithms, so cross-platform performance relies on transfer of Hi-C-trained models
- Probability cutoffs for cell lines other than GM12878 (0.97 CTCF / 0.92 H3K27ac) are only documented in Supplementary Data 2
- Validation rate of loops from Boost-HiC-enhanced down-sampled matrices can be lower depending on the augmentation rate

## Evidence

- [readme] Random Forest classification of chromatin loops: "we present Peakachu, a Random Forest classification framework that predicts chromatin loops from genome-wide contact maps."
- [readme] Leave-one-chromosome-out training rationale: "The model for every chromosome was trained using interactions from all the other 22 chromosomes in the provided bedpe file."
- [discussion] Probability cutoffs for GM12878 models: "we set the cutoff to 0.97 and 0.92 for the CTCF model and the H3K27ac model, respectively."
- [discussion] 11x11 window feature construction and P2LL indicator: "a corresponding 11 × 11 window centered at the interaction is collected from the Hi-C matrix. The ratio of the center pixel to the lower-left quadrant (P2LL) of this window is used as an indicator"
- [discussion] Greedy pooling via anchor peaks and DBSCAN: "define 1D loop anchor regions enriched for highly scored (P > 0.9) pixels, then run DBSCAN between any two connected anchor regions"
- [discussion] Model requires a 10 kb matrix plus positive interaction list: "Fitting a Peakachu model requires two components: a Hi-C matrix binned to 10 kb and an interaction list that defines a positive training set."
