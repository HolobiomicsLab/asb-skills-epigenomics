---
name: cross-species-sequence-dataset-handling
description: Use when building a training/testing corpus from 6mA benchmark datasets
  spanning multiple species (M. musculus + rice), requiring CD-HIT-EST redundancy
  reduction at threshold 0.80 on 41-nt sequences.
when_to_use_negative:
- The task targets a single species only (e.g., an M. musculus-only or rice-only study)
  — use the original species-specific benchmark directly instead.
- The input is already a reduced, non-redundant feature matrix or model-ready encoding
  (e.g., matrix_motif files), not raw sequence benchmarks.
- Sequence lengths differ from 41 nt and the downstream csDMA pipeline (which expects
  41-nt samples) cannot be applied.
tools:
- name: CD-HIT-EST
  role: Reduces sequence redundancy when merging the M. musculus and rice benchmarks,
    run at the rigorous threshold of 0.80 to build the cross-species dataset
- name: csDMA
  role: Downstream consumer of the cross-species dataset; its training, leave-one-out,
    and independent-testing scripts take the resulting positive/negative training
    and test datasets as input
  repo: github.com/NWAFU-LiuLab/csDMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:2231a182c04c673695a9d3b87f69590b5c3eca3a97216121e3c2853334169a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - MEME
  - Python
  - csDMA
  - scikit-learn ExtraTrees
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-species-sequence-dataset-handling

## Summary

Construct a cross-species DNA 6mA benchmark by merging the M. musculus (1,934 pos / 1,934 neg) and rice (880 pos / 880 neg) benchmark datasets into 41-nt samples, then reduce redundancy with CD-HIT-EST at the rigorous threshold of 0.80 and stratified 80/20 train/test split.

## When to use

Use when building a training/testing corpus from 6mA benchmark datasets spanning multiple species (M. musculus + rice), requiring CD-HIT-EST redundancy reduction at threshold 0.80 on 41-nt sequences.

## When NOT to use

- The task targets a single species only (e.g., an M. musculus-only or rice-only study) — use the original species-specific benchmark directly instead.
- The input is already a reduced, non-redundant feature matrix or model-ready encoding (e.g., matrix_motif files), not raw sequence benchmarks.
- Sequence lengths differ from 41 nt and the downstream csDMA pipeline (which expects 41-nt samples) cannot be applied.

## Inputs

- M. musculus DNA 6mA benchmark dataset (1,934 positive / 1,934 negative samples, 41 nt)
- rice DNA 6mA benchmark dataset (880 positive / 880 negative samples, 41 nt)
- CD-HIT-EST redundancy-reduction threshold (0.80)

## Outputs

- Cross-species dataset of 2,768 positive / 2,716 negative 41-nt samples (CD-HIT-EST threshold 0.80)
- Stratified 80% training subset and 20% independent testing subset

## How to apply

Download the M. musculus benchmark (1,934 positive / 1,934 negative) from Feng et al. and the rice benchmark (880 positive / 880 negative) from Chen et al., and combine them into a single 41-nt per-sample corpus. Run CD-HIT-EST across species to remove redundant sequences at the most rigorous threshold of 0.80, yielding 2,768 positive and 2,716 negative samples. Apply stratified random selection so that 80% of samples form the model training set and the remaining 20% form the independent testing set. The resulting dataset feeds the downstream feature-encoding (Motif, Kmer, Binary) and ExtraTrees model-building steps of csDMA.

## Related tools

- **CD-HIT-EST** (Reduces sequence redundancy when merging the M. musculus and rice benchmarks, run at the rigorous threshold of 0.80 to build the cross-species dataset)
- **csDMA** (Downstream consumer of the cross-species dataset; its training, leave-one-out, and independent-testing scripts take the resulting positive/negative training and test datasets as input) — github.com/NWAFU-LiuLab/csDMA

## Evaluation signals

- After CD-HIT-EST at threshold 0.80, the merged corpus contains exactly 2,768 positive and 2,716 negative samples, each 41 nt long.
- The train/test split is stratified: 80% training, 20% independent testing, with positive/negative class balance preserved in both partitions.
- Training and testing subsets are consumable by the csDMA scripts, e.g., 'python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2' runs without schema errors.
- No sequence in the training subset shares >=80% identity with a sequence in the testing subset (the redundancy-reduction invariant).

## Limitations

- CD-HIT-EST threshold choice materially changes dataset size; the 0.80 setting is described as the 'most rigorous threshold', so looser thresholds will retain more (but more redundant) samples.
- csDMA performance on the resulting cross-species dataset was reported as relatively low, compared to its higher performance on the M. musculus dataset.
- csDMA was implemented in Python 2.7, so the downstream scripts consuming this dataset may require a legacy Python environment.

## Evidence

- [intro] the cross-species dataset consists of 2,768 positive samples and 2,716 negative samples with the most rigorous threshold at 0.80, and the length of each sample is 41nt: "the cross-species dataset consists of 2,768 positive samples and 2,716 negative samples with the most rigorous threshold at 0.80, and the length of each sample is 41nt"
- [intro] the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing: "the stratified selection method was used and we random selected 80% samples for model training and the left 20% for independent testing"
- [intro] the CD-HIT-EST software25 with different threshold was used to reduce sequence redundancy: "the CD-HIT-EST software25 with different threshold was used to reduce sequence redundancy"
- [intro] The benchmark dataset includes 1,934 positive samples and 1,934 negative samples.: "The benchmark dataset includes 1,934 positive samples and 1,934 negative samples."
- [readme] csDMA was implemented in python 2.7: "csDMA was implemented in python 2.7"
