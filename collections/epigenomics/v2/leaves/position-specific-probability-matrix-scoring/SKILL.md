---
name: position-specific-probability-matrix-scoring
description: Use when converting MEME-discovered differential motifs around 6mA sites
  into numeric per-sequence features for a classifier, given a FASTA-like training
  set of 41-nt positive/negative samples and motif probability matrices.
when_to_use_negative:
- Input is already an encoded feature table — this skill generates the Motif feature
  matrices from raw 41-nt sequences and MEME probability matrices
- No MEME differential enrichment run or retained motifs (E-value < 0.05) are available
  — there are no probability matrices to score against
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0654
tools:
- name: csDMA
  role: Prediction pipeline whose Motif feature subset (matrix_motif1, matrix_motif2)
    is produced by this scoring step and fed to the ExtraTrees classifier
  repo: https://github.com/liuze-nwafu/csDMA
- name: feature_selection.py
  role: csDMA script that implements the feature encoding scheme and scores each 41-nt
    sample against the motif probability matrices
  repo: https://github.com/liuze-nwafu/csDMA
- name: MEME
  role: Discovers motifs in differential enrichment mode; its position-specific probability
    matrices are the scoring references
- name: scikit-learn ExtraTrees
  role: Downstream classifier consuming the motif score matrices during training and
    LeaveOneOut evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:6c605c5513efa5c264b37a62a37102ae06fe1fd34392717140025821e3070d86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - MEME
  - Python
  - csDMA
  - feature_selection.py
  - scikit-learn ExtraTrees
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# position-specific-probability-matrix-scoring

## Summary

Score each 41-nt 6mA candidate sequence against MEME-derived motif probability matrices (motifs retained at E-value < 0.05) to build per-sample motif score matrices (matrix_motif1, matrix_motif2) used as the Motif feature subset in csDMA's ExtraTrees model.

## When to use

Use when converting MEME-discovered differential motifs around 6mA sites into numeric per-sequence features for a classifier, given a FASTA-like training set of 41-nt positive/negative samples and motif probability matrices.

## When NOT to use

- Input is already an encoded feature table — this skill generates the Motif feature matrices from raw 41-nt sequences and MEME probability matrices
- No MEME differential enrichment run or retained motifs (E-value < 0.05) are available — there are no probability matrices to score against

## Inputs

- 41-nt positive and negative 6mA sample datasets (cross-species: 2,768 positive / 2,716 negative, CD-HIT-EST threshold 0.80)
- Position-specific probability matrices of MEME motifs retained at E-value < 0.05 (matrix_motif1, matrix_motif2 source motifs)

## Outputs

- Per-sample motif score matrices matrix_motif1 and matrix_motif2
- Feature matrix (Motif subset) consumable by leaveoneout.py / csDMA_training.py / classifier_selection.py

## How to apply

First run MEME in differential enrichment mode on the 2,768 positive cross-species 41-nt training samples with the 2,716 negative samples as control, capped at 10 motifs, and filter retained motifs by E-value < 0.05 (e.g., E-values 3.3e-18 and 2.9e-2). Extract each retained motif's position-specific probability matrix from the MEME output. Apply the feature_selection.py encoding script (the csDMA feature-encoding implementation) to score every 41-nt sample against these matrices, writing per-sample motif score matrices matrix_motif1 and matrix_motif2. These motif matrices are consumed downstream by leaveoneout.py, csDMA_training.py, and csDMA_indepedent_testing.py as the Motif feature subset, which combined with Kmer (k=2,3,4; 336 features) and Binary encodings gave the best AUC of 0.871. Validate that the generated matrices match the argument order expected by the downstream scripts: training positive dataset, training negative dataset, matrix_motif1, matrix_motif2.

## Related tools

- **csDMA** (Prediction pipeline whose Motif feature subset (matrix_motif1, matrix_motif2) is produced by this scoring step and fed to the ExtraTrees classifier) — https://github.com/liuze-nwafu/csDMA
- **feature_selection.py** (csDMA script that implements the feature encoding scheme and scores each 41-nt sample against the motif probability matrices) — https://github.com/liuze-nwafu/csDMA
- **MEME** (Discovers motifs in differential enrichment mode; its position-specific probability matrices are the scoring references)
- **scikit-learn ExtraTrees** (Downstream classifier consuming the motif score matrices during training and LeaveOneOut evaluation)

## Examples

```
python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- Only motifs with E-value < 0.05 (e.g., 3.3e-18 and 2.9e-2) contribute matrices — check the retained motif list before scoring
- Generated matrices must be consumable by leaveoneout.py with the exact argument order: training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2
- Feature subset selection with Motif + Kmer + Binary should reproduce the best AUC of 0.871 under 5-fold cross-validation on the training dataset
- Downstream ExtraTrees training must produce csDMA.pkl and normalization.pkl (normalized model file) from these matrices

## Limitations

- Motif scores were computed only for 41-nt samples in the cross-species dataset; performance on the rice and cross-species datasets was relatively low
- Genomic and structural feature encoding schemes are not incorporated in csDMA's encoding, so this skill covers only Motif/Kmer/Binary features
- csDMA was implemented in Python 2.7, which may constrain reproducibility in modern environments

## Evidence

- [intro] the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [abstract] three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [readme] This script implements the feature encoding scheme and can be auto involved in other scripts.: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [readme] Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2: "Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2"
