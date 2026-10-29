---
name: dna-sequence-binary-encoding
description: Use when raw 41-nt DNA sequences (A/C/G/T FASTA records) must be converted
  into fixed-length binary features before classifier training, e.g. reproducing csDMA's
  Binary feature block alongside Motif and Kmer subsets.
when_to_use_negative:
- Input sequences are already encoded as a numeric feature table (e.g. Motif score
  matrix or 336-dim Kmer vectors) rather than raw FASTA nucleotide strings
- Samples are not fixed-length 41-nt DNA reads, or contain characters outside A/C/G/T
  (e.g. RNA or ambiguous IUPAC codes), since the three dichotomies assume exactly
  A, C, G, T
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: csDMA
  role: Python 2.7 toolkit whose feature_selection.py implements the Binary encoding
    scheme; the resulting Binary features feed the ExtraTrees model (csDMA.pkl) together
    with Motif and Kmer features
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:e90e87977a6807e34b18943102eb4c359a81d22171d89cf26e91ae967d0bc3de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - csDMA
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dna-sequence-binary-encoding

## Summary

Encodes each nucleotide of a 41-nt DNA sample into three binary physico-chemical indicator rules (purine/pyrimidine x, A/T-vs-C/G y, A/C-vs-G/T z), producing a 3 × 41 = 123-element feature vector per sequence for csDMA's 6mA prediction model.

## When to use

Use when raw 41-nt DNA sequences (A/C/G/T FASTA records) must be converted into fixed-length binary features before classifier training, e.g. reproducing csDMA's Binary feature block alongside Motif and Kmer subsets.

## When NOT to use

- Input sequences are already encoded as a numeric feature table (e.g. Motif score matrix or 336-dim Kmer vectors) rather than raw FASTA nucleotide strings
- Samples are not fixed-length 41-nt DNA reads, or contain characters outside A/C/G/T (e.g. RNA or ambiguous IUPAC codes), since the three dichotomies assume exactly A, C, G, T

## Inputs

- 41-nt DNA sequence FASTA files: training_positive_dataset and training_negative_dataset (csDMA cross-species dataset, e.g. 2,768 positive / 2,716 negative samples)
- DNA nucleotide strings over the alphabet A, C, G, T

## Outputs

- binary feature matrix with one row per training sample and exactly 123 binary (0/1) columns (41 positions × 3 indicator rules)
- reusable encoding routine in feature_selection.py that can be auto-invoked by other csDMA scripts

## How to apply

Load the training positive and negative 41-nt FASTA datasets (csDMA training_positive_dataset and training_negative_dataset) with a FASTA parser, implemented as a reusable routine in feature_selection.py that other csDMA scripts auto-invoke. For each of the 41 positions compute three indicators: x = 1 for A/G and 0 for C/T (purine vs pyrimidine), y = 1 for A/T and 0 for C/G, and z = 1 for A/C and 0 for G/T. Thus A encodes as (1,1,1), G as (1,0,0), T as (0,1,0), and C as (0,0,1). Concatenate the 41 x-, y-, and z-values into one 123-element binary vector per sample and stack all samples into the binary feature block. In csDMA this block is combined with the Motif and Kmer subsets (Kmer with k = 2, 3, 4 giving 336 features) and fed to the ExtraTrees classifier, where Motif+Kmer+Binary was the best feature-subset combination (AUC 0.871).

## Related tools

- **csDMA** (Python 2.7 toolkit whose feature_selection.py implements the Binary encoding scheme; the resulting Binary features feed the ExtraTrees model (csDMA.pkl) together with Motif and Kmer features) — https://github.com/liuze-nwafu/csDMA

## Evaluation signals

- Output matrix has exactly one row per input sample and 123 columns, all values in {0, 1}, with no missing positions
- Per-position code consistency: every A maps to (1,1,1), G to (1,0,0), T to (0,1,0), C to (0,0,1), so x + y + z = 3 iff the base is A
- Each position's 3-bit codeword is unique across the four bases (no colliding encodings)
- When combined with Motif and Kmer subsets, the downstream ExtraTrees 5-fold cross-validation should reach the paper's best AUC of 0.871 (and 0.878 training / 0.893 independent-test AUC for the full model)

## Limitations

- Performance of the full csDMA model was relatively low on the rice and cross-species datasets compared to the M. musculus dataset, so binary-encoded features alone do not guarantee strong cross-species discrimination
- The paper notes that additional feature types (genomic and structural features) were not yet incorporated and are planned to improve csDMA, i.e. the Binary scheme is a simple physico-chemical dichotomy
- The README gives no CLI usage line for feature_selection.py; it is designed to be auto-invoked by other csDMA scripts (e.g. csDMA_training.py) rather than run standalone

## Evidence

- [abstract] three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [intro] best feature-subset AUC: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [readme] feature_selection.py role: "feature_selection.py<br>#This script implements the feature encoding scheme and can be auto involved in other scripts."
