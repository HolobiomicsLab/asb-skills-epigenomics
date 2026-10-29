---
name: cross-species-dna-6ma-dataset-handling
description: Use when raw labeled FASTA sequences (e.g., the cross-species DNA 6mA
  benchmark of 2,768 positive / 2,716 negative 41-nt samples) must be converted into
  fixed-length numeric k-mer frequency features for machine-learning classification.
when_to_use_negative:
- Input is already a numeric feature matrix (e.g., precomputed matrix_motif or matrix_binary
  files) rather than raw FASTA sequences
- Sequences are not fixed-length DNA segments around 6mA sites (the csDMA cross-species
  samples are 41-nt); variable-length or non-DNA input breaks the window-count normalization
- The goal is classifier training rather than feature encoding — use csDMA_training.py
  or classifier_selection.py with the already-encoded matrices instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0697
- http://edamontology.org/topic_3318
tools:
- name: csDMA
  role: Provides feature_selection.py, which implements the k=2,3,4 k-mer relative-frequency
    encoding scheme and is auto-invoked by the other csDMA scripts
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:4ce7c816d805755d89f388d4f8328ca1649a7dfbe9f21a46a04a403b55b71714
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0697
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-species-dna-6ma-dataset-handling

## Summary

Implements the csDMA K-mer pattern frequency encoding that converts 41-nt DNA 6mA candidate sequences into numerical feature vectors by computing relative frequencies of dinucleotide, trinucleotide, and tetranucleotide patterns (k=2,3,4 → 4^2+4^3+4^4 = 336 features per sample).

## When to use

Use when raw labeled FASTA sequences (e.g., the cross-species DNA 6mA benchmark of 2,768 positive / 2,716 negative 41-nt samples) must be converted into fixed-length numeric k-mer frequency features for machine-learning classification.

## When NOT to use

- Input is already a numeric feature matrix (e.g., precomputed matrix_motif or matrix_binary files) rather than raw FASTA sequences
- Sequences are not fixed-length DNA segments around 6mA sites (the csDMA cross-species samples are 41-nt); variable-length or non-DNA input breaks the window-count normalization
- The goal is classifier training rather than feature encoding — use csDMA_training.py or classifier_selection.py with the already-encoded matrices instead

## Inputs

- Labeled positive and negative training FASTA files of 41-nt DNA sequences (cross-species DNA 6mA benchmark: 2,768 positive / 2,716 negative samples)
- Parameter k set (k=2,3,4 per csDMA)

## Outputs

- K-mer feature matrix (e.g., matrix_kmer.csv) with exactly 336 k-mer frequency columns plus sample labels, one row per sequence

## How to apply

Load the labeled positive/negative training FASTA files and enumerate every contiguous dinucleotide (16), trinucleotide (64), and tetranucleotide (256) window in each 41-nt sequence. For each k-mer pattern, compute its relative frequency as its occurrence count divided by the number of windows of that k-mer length in the sequence (e.g., 40 windows for k=2). Concatenate the 16+64+256 = 336 relative-frequency values in a fixed order into a single feature vector per sample. Assemble the per-sample vectors with their class labels into a k-mer feature matrix and write it out (e.g., matrix_kmer.csv); the csDMA feature_selection.py script implements this encoding and is auto-invoked by the other scripts.

## Related tools

- **csDMA** (Provides feature_selection.py, which implements the k=2,3,4 k-mer relative-frequency encoding scheme and is auto-invoked by the other csDMA scripts) — https://github.com/liuze-nwafu/csDMA

## Examples

```
from itertools import product; X=[[s.count(''.join(p))/(len(s)-k+1) for k in (2,3,4) for p in product('ACGT',repeat=k)] for s in seqs]; pd.DataFrame(X).to_csv('matrix_kmer.csv')
```

## Evaluation signals

- The output matrix contains exactly 336 k-mer feature columns for every sample, matching the reported 4^2+4^3+4^4 = 336 encoding
- Column-count decomposition check: 16 dinucleotide + 64 trinucleotide + 256 tetranucleotide frequency columns are present
- Every feature value lies in [0, 1] and each k-block's values are consistent with counts divided by the number of windows of that length (e.g., 40 for k=2 in a 41-nt sequence)
- Row count equals the number of input FASTA records, and labels are preserved for downstream 5-fold cross-validation
- Feeding the matrix into classifier_selection.py's 5-fold CV (RandomForest, GradientBoosting, AdaBoost, ExtraTrees, SVM) runs without shape errors

## Limitations

- Only k=2,3,4 was evaluated in csDMA; the Kmer subset alone is insufficient for best performance — Motif + Kmer + Binary combined (AUC 0.871) was the best feature combination
- csDMA performance on the rice and cross-species datasets was relatively low compared to the M. musculus dataset
- Genomic and structural feature encoding schemes are not yet incorporated into csDMA

## Evidence

- [intro] k-mer nucleotide pattern frequency encoding: "we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding."
- [intro] cross-species dataset composition: "the cross-species dataset consists of 2,768 positive samples and 2,716 negative samples with the most rigorous threshold at 0.80, and the length of each sample is 41nt"
- [abstract] role of the encoding in the csDMA pipeline: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [readme] feature_selection.py implements the encoding: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
