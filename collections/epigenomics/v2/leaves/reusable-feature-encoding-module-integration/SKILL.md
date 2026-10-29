---
name: reusable-feature-encoding-module-integration
description: Use when converting 41-nt DNA sequences (FASTA) into the csDMA Binary
  feature block, or when a feature-encoding routine must live in one shared script
  (feature_selection.py) that classifier-selection, training, and testing scripts
  can auto-invoke.
when_to_use_negative:
- Input is already a numeric feature matrix (e.g. precomputed Motif or Kmer matrices
  like matrix_motif1/matrix_motif2) — no sequence-level binary encoding is needed.
- Sequences are not 41 nt (e.g. the k-mer scheme with k=2,3,4 or other read lengths)
  — the 3 × 41 = 123 feature layout assumes exactly 41 positions.
- The goal is per-sample motif scores or k-mer nucleotide pattern counts (336 features
  for k=2,3,4) rather than per-nucleotide dichotomy indicators.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_3474
tools:
- name: csDMA
  role: Source toolkit whose feature_selection.py hosts the reusable Binary encoding
    routine invoked by the classifier-selection, training, and independent-testing
    scripts
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:39035936b9f34fd3eb5dcc128e10074d089cabb5c83ffd7db515ce3ab4d5adc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# reusable-feature-encoding-module-integration

## Summary

Implements csDMA's Binary code encoding — 3 purine/pyrimidine-style dichotomy indicators (x, y, z) per nucleotide position over a 41-nt sequence, yielding 3 × 41 = 123 binary features — as a reusable routine inside the csDMA feature encoding script (feature_selection.py) so other scripts can auto-invoke it.

## When to use

Use when converting 41-nt DNA sequences (FASTA) into the csDMA Binary feature block, or when a feature-encoding routine must live in one shared script (feature_selection.py) that classifier-selection, training, and testing scripts can auto-invoke.

## When NOT to use

- Input is already a numeric feature matrix (e.g. precomputed Motif or Kmer matrices like matrix_motif1/matrix_motif2) — no sequence-level binary encoding is needed.
- Sequences are not 41 nt (e.g. the k-mer scheme with k=2,3,4 or other read lengths) — the 3 × 41 = 123 feature layout assumes exactly 41 positions.
- The goal is per-sample motif scores or k-mer nucleotide pattern counts (336 features for k=2,3,4) rather than per-nucleotide dichotomy indicators.

## Inputs

- training_positive_dataset (41-nt DNA 6mA positive sequences, FASTA)
- training_negative_dataset (41-nt non-6mA negative sequences, FASTA)

## Outputs

- binary_features.csv: one row per training sample with exactly 123 binary (0/1) columns (41 positions × 3 indicators), no missing positions
- Binary feature block consumable by classifier_selection.py, csDMA_training.py, and csDMA_indepedent_testing.py via auto-invocation of feature_selection.py

## How to apply

Load the training positive and negative 41-nt DNA sequence datasets with a FASTA parser, then for each of the 41 nucleotide positions compute three binary indicators: x = 1 if the base is A or G (purine) and 0 if C or T (pyrimidine); y = 1 if the base is A or T and 0 if C or G (A/T vs C/G); z = 1 if the base is A or C and 0 if G or T (A/C vs G/T). Equivalently, per the paper, A is encoded (1,1,1), C (1,0,0), G (0,1,0), and T (0,0,1). Concatenate the 41 x-, y-, and z-values into a single 123-element vector (41 positions × 3 rules) per sample and stack all samples into the binary feature block. Implement this as a reusable routine inside feature_selection.py — which per the README 'implements the feature encoding scheme and can be auto involved in other scripts' — so it integrates with the Motif and Kmer blocks that together with Binary gave the paper's best feature-subset AUC of 0.871.

## Related tools

- **csDMA** (Source toolkit whose feature_selection.py hosts the reusable Binary encoding routine invoked by the classifier-selection, training, and independent-testing scripts) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python feature_selection.py training_positive_dataset training_negative_dataset  # Binary block (3x41=123 features) is built inside and auto-invoked by other csDMA scripts
```

## Evaluation signals

- binary_features.csv contains one row per training sample (2,768 positive / 2,716 negative for the cross-species dataset) with exactly 123 binary (0/1) columns.
- No missing positions: every row covers all 41 nucleotide positions across the three indicator rules (x, y, z).
- Per-position encoding is self-consistent: each base maps uniquely to (x,y,z) — A=(1,1,1), C=(1,0,0), G=(0,1,0), T=(0,0,1) — so reversed lookup recovers the original sequence.
- Downstream reproduction check: the Motif + Kmer + Binary feature combination should yield AUC ≈ 0.871 in the 5-fold cross-validation feature-subset selection on the training dataset.
- The routine requires no manual wiring: other scripts (classifier_selection.py, csDMA_training.py, csDMA_indepedent_testing.py) auto-invoke it via feature_selection.py.

## Limitations

- The 123-element layout is fixed to 41-nt samples; it does not generalize to other sequence lengths without redefinition.
- csDMA was implemented in Python 2.7, so the encoding module inherits that runtime dependency.
- Binary encoding alone is not sufficient: csDMA's best performance (AUC 0.871) required combining it with the Motif and Kmer feature subsets, and performance was relatively low on the rice and cross-species datasets.

## Evidence

- [full_text] The Binary encoding scheme generates a vector with 3 × 41 = 123 elements, characterizing each nucleotide A, C, G, or T with (1,1,1), (0,0,1), (1,0,0), or (0,1,0) respectively.: [evidence span withheld — anchor status: not_found]
- [readme] feature_selection.py implements the feature encoding scheme and can be auto-invoked by other csDMA scripts.: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [results] the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [abstract] Three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
