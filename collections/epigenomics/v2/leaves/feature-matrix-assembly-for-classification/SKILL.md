---
name: feature-matrix-assembly-for-classification
description: Use when raw 41-nt DNA 6mA candidate sequences (positive/negative FASTA
  sets) must be converted into a numeric feature matrix to train or test a machine-learning
  classifier.
when_to_use_negative:
- Input is already a numeric feature matrix or feature table rather than raw 41-nt
  nucleotide sequences.
- Sequences are not 41 nt long — the Binary scheme hard-codes 3 x 41 = 123 features,
  so other lengths break the encoding.
- The goal is feature engineering with genomic or structural properties, which csDMA
  does not yet implement (flagged as future work).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: csDMA
  role: Python 2.7 package whose feature_selection.py implements the Motif, Kmer,
    and Binary encoding schemes and is auto-invoked by the training, classifier-selection,
    and independent-testing scripts
  repo: github.com/NWAFU-LiuLab/csDMA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:cc3eaf645c4a45fe3702181626a33fa23c67cc265f1ab640f311509f89258af5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# feature-matrix-assembly-for-classification

## Summary

Assemble the csDMA feature matrix for DNA 6mA classification by combining three encoding schemes — Motif, Kmer (k=2,3,4; 336 features), and Binary code (3 x 41 = 123 features) — applied to 41-nt FASTA sequence datasets, since Motif+Kmer+Binary was the best feature subset (AUC 0.871) feeding the ExtraTrees model.

## When to use

Use when raw 41-nt DNA 6mA candidate sequences (positive/negative FASTA sets) must be converted into a numeric feature matrix to train or test a machine-learning classifier.

## When NOT to use

- Input is already a numeric feature matrix or feature table rather than raw 41-nt nucleotide sequences.
- Sequences are not 41 nt long — the Binary scheme hard-codes 3 x 41 = 123 features, so other lengths break the encoding.
- The goal is feature engineering with genomic or structural properties, which csDMA does not yet implement (flagged as future work).

## Inputs

- training_positive_dataset and training_negative_dataset (41-nt DNA 6mA FASTA files)
- test_positive_dataset and test_negative_dataset (independent-test FASTA files)
- matrix_motif1 and matrix_motif2 (motif score matrix feature inputs)

## Outputs

- binary_features.csv (one row per sample with exactly 123 binary 0/1 columns)
- combined feature matrix (Motif + Kmer 336 + Binary 123 blocks) consumed by the classifier scripts
- csDMA.pkl (trained ExtraTrees model file) and normalization.pkl (feature normalization file)

## How to apply

Load the training positive and negative 41-nt FASTA datasets with a FASTA parser and encode each sequence with three schemes inside feature_selection.py, a routine that can be auto-invoked by other scripts. Kmer encoding with k=2,3,4 yields 42+43+44=336 nucleotide-pattern features; Binary encoding maps each nucleotide to 3 indicator bits — (1,1,1) for A, (0,0,1) for C, (1,0,0) for G, (0,1,0) for T — giving 3 x 41 = 123 binary features per sample, and Motif contributes motif score matrix features (matrix_motif1, matrix_motif2 inputs). Concatenate the per-sample feature blocks into the full matrix and pass it together with the FASTA datasets to classifier_selection.py (RandomForest, GradientBoosting, AdaBoost, ExtraTrees, SVM, ensemble) or csDMA_training.py. Decision point: the paper found Motif+Kmer+Binary gave the best AUC of 0.871 by 5-fold cross-validation, and ExtraTrees was the best classifier, so keep all three blocks and train with ExtraTrees (outputs csDMA.pkl plus normalization.pkl for input scaling).

## Related tools

- **csDMA** (Python 2.7 package whose feature_selection.py implements the Motif, Kmer, and Binary encoding schemes and is auto-invoked by the training, classifier-selection, and independent-testing scripts) — github.com/NWAFU-LiuLab/csDMA

## Examples

```
python csDMA_training.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2 csDMA.pkl normalization.pkl
```

## Evaluation signals

- binary_features.csv contains one row per training sample with exactly 123 binary (0/1) columns and no missing positions.
- Kmer block dimensionality is 42+43+44 = 336 columns for k = 2, 3, 4.
- Motif+Kmer+Binary subset reproduces the paper's best 5-fold cross-validation AUC of 0.871.
- Trained model yields csDMA.pkl and normalization.pkl; ExtraTrees outperforms RandomForest, GradientBoosting, AdaBoost, and SVM on the training and independent test sets (AUC 0.893 on the independent set).
- Feature matrix dimensions match the number of samples in the input FASTA files (e.g., 2,768 positive / 2,716 negative for the cross-species dataset).

## Limitations

- csDMA performance is relatively low on the rice and cross-species datasets compared to the M. musculus dataset.
- The Binary scheme as specified supports only A, C, G, and T — ambiguous nucleotide codes are not handled.
- Only sequence-based encodings are used; genomic and structural feature encoding schemes are not yet incorporated.

## Evidence

- [abstract] three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [intro] best feature-subset combination: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [readme] feature_selection.py purpose: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [readme] training invocation: "Usage: python csDMA_training.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2 model_file scale_file"
