---
name: nucleotide-chemical-dichotomy-encoding
description: Use when converting fixed-length 41-nt DNA sequences (A/C/G/T) into per-position
  binary features for the csDMA 6mA classifier, when Motif and Kmer features alone
  underperform, or when a purely chemical-dichotomy encoding of nucleotides is needed.
when_to_use_negative:
- Input sequences are not 41 nt or contain ambiguous bases beyond A/C/G/T, since the
  three dichotomy rules only partition A, C, G, and T.
- Input is already a numeric feature table (e.g. existing Kmer/Motif matrix or precomputed
  binary features) — re-encoding would duplicate features rather than add the dichotomy
  representation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: csDMA
  role: The host tool whose feature encoding scheme implements the Binary (chemical
    dichotomy) encoding as part of feature_selection.py; trained ExtraTrees model
    consumes the resulting 123-feature block.
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:ed5202ead674417bd87438040823266dd33f37fc4516f67549af688b3310d450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nucleotide-chemical-dichotomy-encoding

## Summary

Binary nucleotide encoding scheme from csDMA that turns each nucleotide of a 41-nt DNA 6mA sample into three chemical-dichotomy indicators (purine/pyrimidine, A/T vs C/G, A/C vs G/T), producing a 3 × 41 = 123-element feature vector per sequence. It is one of the three feature blocks (Motif, Kmer, Binary) whose combination gave csDMA's best feature-subset AUC of 0.871.

## When to use

Use when converting fixed-length 41-nt DNA sequences (A/C/G/T) into per-position binary features for the csDMA 6mA classifier, when Motif and Kmer features alone underperform, or when a purely chemical-dichotomy encoding of nucleotides is needed.

## When NOT to use

- Input sequences are not 41 nt or contain ambiguous bases beyond A/C/G/T, since the three dichotomy rules only partition A, C, G, and T.
- Input is already a numeric feature table (e.g. existing Kmer/Motif matrix or precomputed binary features) — re-encoding would duplicate features rather than add the dichotomy representation.

## Inputs

- 41-nt DNA sequence FASTA files (training_positive_dataset, training_negative_dataset, and independent test positives/negatives) containing only A, C, G, T bases
- Feature subset combination selector (Motif + Kmer + Binary) when used inside the csDMA pipeline

## Outputs

- A binary feature table (e.g. binary_features.csv) with one row per sample and exactly 123 binary (0/1) columns (41 positions × 3 dichotomy rules)
- A feature matrix block concatenated with Motif and Kmer features for classifier training/testing

## How to apply

Load the 41-nt training positive/negative FASTA datasets (csDMA training_positive_dataset / training_negative_dataset) with a FASTA parser, then for each of the 41 positions compute three binary indicators: x = 1 for A or G and 0 for C or T (purine vs pyrimidine), y = 1 for A or T and 0 for C or G (A/T vs C/G), and z = 1 for A or C and 0 for G or T (A/C vs G/T). Concatenate the 41 x-, y-, and z-values into a single 123-element (0/1) vector per sample and stack all samples into the binary feature block. Implement this as a reusable routine inside feature_selection.py so other csDMA scripts (classifier_selection.py, csDMA_training.py, csDMA_indepedent_testing.py) can auto-invoke it. Combine the resulting Binary block with the Motif and Kmer subsets, since that combination produced the paper's best AUC of 0.871; the downstream ExtraTrees model then reached AUC 0.893 on the independent test set.

## Related tools

- **csDMA** (The host tool whose feature encoding scheme implements the Binary (chemical dichotomy) encoding as part of feature_selection.py; trained ExtraTrees model consumes the resulting 123-feature block.) — https://github.com/liuze-nwafu/csDMA

## Examples

```
x=[1 if b in 'AG' else 0 for b in seq]; y=[1 if b in 'AT' else 0 for b in seq]; z=[1 if b in 'AC' else 0 for b in seq]; feats=x+y+z
```

## Evaluation signals

- binary_features.csv contains exactly one row per training sample with exactly 123 columns, all values in {0,1}, and no missing positions.
- Each 41-position window maps A→(1,1,1), C→(0,0,1)... i.e. spot-check: A gives x=1,y=1,z=1; G gives x=1,y=0,z=0; T gives x=0,y=1,z=0 per the A/C vs G/T rule; C gives x=0,y=0,z=1.
- Row count equals the dataset size (e.g. 2,768 positives + 2,716 negatives in the cross-species dataset at CD-HIT-EST threshold 0.80).
- Adding the Binary block to Motif + Kmer reproduces the paper's best feature-subset AUC of 0.871 under 5-fold cross-validation with ExtraTrees.
- The routine in feature_selection.py is auto-invoked by csDMA_training.py and csDMA_indepedent_testing.py without duplicating encoding logic.

## Limitations

- Performance degrades on species datasets other than the training distribution: csDMA showed relatively low performance on the rice and cross-species datasets compared to M. musculus.
- Genomic and structural feature encoding schemes were not yet incorporated, so this purely sequence-dichotomy encoding misses those signals.
- csDMA was implemented in Python 2.7; the encoding routine inherits that runtime assumption.

## Evidence

- [results] Binary encoding per-nucleotide codes: [evidence span withheld — anchor status: not_found]
- [intro] Best feature-subset combination: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [readme] Encoding script auto-invocation: "This script implements the feature encoding scheme and can be auto involved in other scripts."
- [abstract] Three encoding schemes: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
