---
name: super-enhancer-typical-enhancer-sequence-classification
description: Use when the goal is to distinguish super-enhancers from typical enhancers
  from raw DNA sequences (FASTA) in a given cell line (e.g. mESC), or to reproduce
  the DeepSE-specific model with epochs=80, learning_rate=0.001.
when_to_use_negative:
- Input is already a feature matrix or epigenomic (ChIP-seq) data rather than raw
  DNA sequences — DeepSE is specifically designed for sequence-only input
- You need a cross-cell-line or cross-species general model rather than a cell-line
  specific one — use the train_integrative1/2/3.py workflows instead of train_specific.py
- The cell line's SE/TE sequences are not in FASTA format and cannot be placed in
  the directory structure expected by get_datavec.py
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DeepSE
  role: Two-layer CNN model that distinguishes super-enhancers from typical enhancers
    using DNA sequence feature embeddings
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Provides the pre-trained k-mer feature vectors (mm9/hg19 embeddings) used
    to encode SE/TE DNA sequences into feature matrices
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Framework in which the two-layer CNN of model.py is implemented and trained
- name: Python
  role: Runtime environment for DeepSE (requires Python >=3.6); all pipeline scripts
    (get_datavec.py, train_specific.py, LogUtils.py) are Python CLI tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:08f51f0382e5d5c6dc8d056a65b3ff78452684aa9e98b9dad6248c081280f801
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/QiaoyingJi/DeepSE
  repo_url: https://github.com/QiaoyingJi/DeepSE
  tools:
  - DeepSE
  - Keras
  - Python
  - dna2vec
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# super-enhancer-typical-enhancer-sequence-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the DeepSE approach: a two-layer CNN in Keras that classifies super-enhancers (SEs) versus typical enhancers (TEs) from DNA sequence alone, using pre-trained dna2vec k-mer feature embeddings and no ChIP-seq data.

## When to use

Use when the goal is to distinguish super-enhancers from typical enhancers from raw DNA sequences (FASTA) in a given cell line (e.g. mESC), or to reproduce the DeepSE-specific model with epochs=80, learning_rate=0.001.

## When NOT to use

- Input is already a feature matrix or epigenomic (ChIP-seq) data rather than raw DNA sequences — DeepSE is specifically designed for sequence-only input
- You need a cross-cell-line or cross-species general model rather than a cell-line specific one — use the train_integrative1/2/3.py workflows instead of train_specific.py
- The cell line's SE/TE sequences are not in FASTA format and cannot be placed in the directory structure expected by get_datavec.py

## Inputs

- DNA sequence files of super-enhancers (SEs) and typical enhancers (TEs) in FASTA format, organized by cell name
- Pre-trained dna2vec k-mer embedding matrix (mouse mm9 or human hg19; or a custom dna2vec embedding)
- Encoded cell line datasets available from the repository (e.g. mESC, mESC_constituent, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)

## Outputs

- Encoded k-mer feature matrices written to ./data/<cell_name> (e.g. ./data/mESC)
- Trained cell-line specific Keras model (DeepSE-specific) for the chosen cell line
- Log files recording training information (via LogUtils.py)
- Evaluation metrics (e.g. accuracy/AUC) on held-out SE/TE test data

## How to apply

Install the third-party packages from requirements.txt into a Python >=3.6 environment. Encode the cell line's SE/TE DNA sequences (FASTA format) into k-mer feature matrices with the pre-trained mouse (mm9) dna2vec embedding by running 'python get_datavec.py mESC mm9', which writes encoded matrices to ./data/mESC. Instantiate the two-layer CNN defined in model.py with Keras and train the cell-line specific model via 'python train_specific.py mESC 0.001 80' (learning_rate=0.001, epochs=80 as in the paper's demo), using LogUtils.py so training output is printed to console and recorded in log files. Finally, evaluate the trained model on held-out SE/TE data and save an accuracy/AUC metrics record alongside the Keras model. The rationale is that dna2vec sequence embeddings alone carry enough signal to separate SEs from TEs without ChIP-seq data.

## Related tools

- **DeepSE** (Two-layer CNN model that distinguishes super-enhancers from typical enhancers using DNA sequence feature embeddings) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Provides the pre-trained k-mer feature vectors (mm9/hg19 embeddings) used to encode SE/TE DNA sequences into feature matrices) — https://ngpatrick.com/dna2vec/
- **Keras** (Framework in which the two-layer CNN of model.py is implemented and trained)
- **Python** (Runtime environment for DeepSE (requires Python >=3.6); all pipeline scripts (get_datavec.py, train_specific.py, LogUtils.py) are Python CLI tools)

## Examples

```
python get_datavec.py mESC mm9; python train_specific.py mESC 0.001 80
```

## Evaluation signals

- Encoded feature matrices for the chosen cell line exist at ./data/<cell_name> (e.g. ./data/mESC) after get_datavec.py completes
- Training console output is mirrored into log files produced by LogUtils.py
- The trained model uses the demo hyperparameters learning_rate=0.001 and epochs=80 for mESC
- The trained model achieves accuracy/AUC on held-out mESC SE/TE data that can be saved and compared against the paper's reported performance (DeepSE outperformed state-of-the-art methods)
- Runtime environment satisfies Python >=3.6 and all packages from requirements.txt import without error

## Limitations

- Pre-trained dna2vec embeddings are only provided for mouse (mm9) and human (hg19); other genomes require training a custom dna2vec model
- The README's installation instruction text ('pip install requirements.txt') and the demo text are inconsistent about learning_rate (0.01 vs 0.001); the demo command and paper use 0.001
- The discussion-section content is unavailable for extraction, so cell-type generalization caveats cannot be fully traced

## Evidence

- [readme] To predict SEs in cell-line specific way, you can type the command line like follows: python train_specific.py cell_name learning_rate epochs: [evidence span withheld — anchor status: not_found]
- [methods] python get_datavec.py cell_name genome ... And you can find the encoded matrixs of mESC in the file of path './data/mESC'.: [evidence span withheld — anchor status: not_found]
