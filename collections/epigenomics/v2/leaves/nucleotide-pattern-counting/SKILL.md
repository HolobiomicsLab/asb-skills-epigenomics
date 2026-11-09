---
name: nucleotide-pattern-counting
description: Use when converting labeled DNA sequences (e.g., FASTA of 41-nt 6mA positives/negatives)
  into a fixed-length numeric feature matrix via k-mer relative-frequency encoding
  before classifier training.
when_to_use_negative:
- Input is already a numeric feature matrix or the downstream model expects motif/binary
  encodings rather than k-mer frequencies
- Sequences vary widely in length and window-count normalization by k-mer length would
  distort relative frequencies (csDMA uses fixed 41-nt samples)
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: csDMA
  role: Provides feature_selection.py implementing the K-mer (k=2,3,4) relative-frequency
    encoding scheme invoked by the training and classifier-selection scripts
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:ede02a66c31805e0df2844e0e4e15cb0ba14f215edc3709fb2a728379a1abc99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nucleotide-pattern-counting

## Summary

Encodes fixed-length DNA sequences (e.g., 41-nt 6mA benchmark samples) by computing relative frequencies of contiguous k-mer nucleotide patterns; in csDMA, k=2,3,4 yields 16+64+256=336 features per sample.

## When to use

Use when converting labeled DNA sequences (e.g., FASTA of 41-nt 6mA positives/negatives) into a fixed-length numeric feature matrix via k-mer relative-frequency encoding before classifier training.

## When NOT to use

- Input is already a numeric feature matrix or the downstream model expects motif/binary encodings rather than k-mer frequencies
- Sequences vary widely in length and window-count normalization by k-mer length would distort relative frequencies (csDMA uses fixed 41-nt samples)

## Inputs

- Labeled training FASTA datasets of 41-nt sequences (cross-species DNA 6mA benchmark: 2,768 positive / 2,716 negative samples)
- Independent test FASTA datasets (20% held-out stratified split)
- Choice of k values (csDMA uses k=2, 3, 4)

## Outputs

- K-mer feature matrix with exactly 336 relative-frequency columns per sample plus sample labels (e.g., matrix_kmer.csv)
- Encoded feature vectors consumable by classifier_selection.py and csDMA_training.py

## How to apply

Enumerate every contiguous dinucleotide (16 patterns), trinucleotide (64), and tetranucleotide (256) window in each 41-nt sequence, computing each k-mer's relative frequency as its occurrence count divided by the number of windows of that length. Set k to 2, 3, and 4 as in csDMA, concatenating the 336 relative-frequency values into one feature vector per sample. Implement the encoding in the feature_selection.py script so it can be auto-invoked by classifier_selection.py and csDMA_training.py. Assemble the vectors (with sample labels) into a k-mer feature matrix that is then combined with Motif and Binary subsets — csDMA found Motif+Kmer+Binary gave the best AUC of 0.871 with ExtraTrees.

## Related tools

- **csDMA** (Provides feature_selection.py implementing the K-mer (k=2,3,4) relative-frequency encoding scheme invoked by the training and classifier-selection scripts) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python feature_selection.py  # invoked from csDMA_training.py: python csDMA_training.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2 model_file scale_file
```

## Evaluation signals

- Output matrix contains exactly 336 k-mer feature columns per sample, consistent with 4^2+4^3+4^4
- Each feature value is a relative frequency in [0, 1]; per-k feature blocks are 16, 64, and 256 columns
- Encoding runs end-to-end within feature_selection.py and is auto-invoked by csDMA_training.py and classifier_selection.py
- Downstream ExtraTrees 5-fold CV on the combined Motif+Kmer+Binary matrix reaches AUC ≈ 0.871 (0.878 on training data per abstract)

## Limitations

- csDMA performance on the rice and cross-species datasets was relatively low, suggesting k-mer/motif/binary encodings may not capture species-specific signals
- Genomic and structural feature encoding schemes are not incorporated; k-mer frequencies alone omit positional/structural context
- csDMA was implemented in Python 2.7, which may require an older environment to reproduce the encoding

## Evidence

- [intro] k-mer feature count: "we set k as 2, 3, 4, and extracted 42 + 43 + 44 = 336 kinds of k-mer nucleotide patterns for feature encoding."
- [methods] feature_selection.py role: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [intro] best feature combination: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [intro] cross-species dataset shape: "the cross-species dataset consists of 2,768 positive samples and 2,716 negative samples with the most rigorous threshold at 0.80, and the length of each sample is 41nt"
