---
name: pretrained-embedding-vectorization
description: Use when raw DNA sequences in FASTA format must be turned into fixed
  feature-vector inputs for a deep learning model, and a species-matched pre-trained
  dna2vec embedding matrix (hg19 or mm9) is available.
when_to_use_negative:
- The encoded cell data are already available — the authors uploaded all encoded matrices
  used in their experiments (mESC_constituent through U87), so re-encoding is unnecessary
- The sequences belong to a genome build other than hg19 or mm9 and no custom dna2vec
  matrix has been trained
- The downstream model expects raw one-hot or k-mer count encodings rather than dna2vec
  feature embeddings
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: dna2vec
  role: Provides the pre-trained k-mer feature vectors used to convert DNA sequences
    into dense embedding matrices; custom matrices can be trained via https://ngpatrick.com/dna2vec/
  repo: https://ngpatrick.com/dna2vec/
- name: get_datavec.py
  role: Encoding script invoked as 'python get_datavec.py cell_name genome' to convert
    FASTA SE/TE sequences into embedding matrices using the selected pre-trained matrix
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Python
  role: Runtime environment (Python >=3.6) in which the encoding script is executed
  repo: https://github.com/QiaoyingJi/DeepSE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:d38456663e3dedd0f24c0db8f7042b6aab84e8a6f151767670bf381d5fc74c6d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/QiaoyingJi/DeepSE
  tools:
  - DeepSE
  - Python
  - dna2vec
  - get_datavec.py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pretrained-embedding-vectorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts raw enhancer DNA sequences (FASTA) into dense numerical feature matrices using pre-trained dna2vec k-mer embedding matrices (human hg19 or mouse mm9), producing CNN-ready inputs for the DeepSE super-enhancer vs typical-enhancer classifier.

## When to use

Use when raw DNA sequences in FASTA format must be turned into fixed feature-vector inputs for a deep learning model, and a species-matched pre-trained dna2vec embedding matrix (hg19 or mm9) is available.

## When NOT to use

- The encoded cell data are already available — the authors uploaded all encoded matrices used in their experiments (mESC_constituent through U87), so re-encoding is unnecessary
- The sequences belong to a genome build other than hg19 or mm9 and no custom dna2vec matrix has been trained
- The downstream model expects raw one-hot or k-mer count encodings rather than dna2vec feature embeddings

## Inputs

- SE and TE DNA sequence files in FASTA format, organized in directories by cell name
- Pre-trained dna2vec embedding matrix (human hg19 or mouse mm9), or a custom-trained dna2vec matrix

## Outputs

- Encoded feature-vector matrices per cell-line dataset, written to './data/<cell_name>' (e.g. './data/mESC'), suitable as CNN inputs

## How to apply

Organize the SE and TE sequence files in FASTA format into directories named by cell line (e.g. mESC, myotube, H2171). Select the embedding matrix matching the genome build: human hg19 for H2171, MM1.S, and U87; mouse mm9 for mESC_constituent, mESC, myotube, macrophage, pro-B cell, and Th-cell; alternatively train a custom dna2vec matrix from https://ngpatrick.com/dna2vec/. Run 'python get_datavec.py cell_name genome' for each cell-line/genome pair (e.g. 'python get_datavec.py mESC mm9'), which converts each TE/SE sequence into k-mer feature vectors. Collect the resulting encoded matrices from './data/<cell_name>' (e.g. './data/mESC') as the numerical inputs to the two-layer Keras CNN. The rationale is that dna2vec embeddings capture k-mer sequence patterns that let the CNN distinguish SEs from TEs using sequence information alone, without ChIP-seq data.

## Related tools

- **dna2vec** (Provides the pre-trained k-mer feature vectors used to convert DNA sequences into dense embedding matrices; custom matrices can be trained via https://ngpatrick.com/dna2vec/) — https://ngpatrick.com/dna2vec/
- **get_datavec.py** (Encoding script invoked as 'python get_datavec.py cell_name genome' to convert FASTA SE/TE sequences into embedding matrices using the selected pre-trained matrix) — https://github.com/QiaoyingJi/DeepSE
- **Python** (Runtime environment (Python >=3.6) in which the encoding script is executed) — https://github.com/QiaoyingJi/DeepSE

## Examples

```
python get_datavec.py mESC mm9
```

## Evaluation signals

- The encoding script completes without error for each cell-line/genome pair
- Encoded matrix files exist in the expected output path './data/<cell_name>' (e.g. './data/mESC' after encoding mESC with mm9)
- The correct embedding matrix was used per species: hg19 for H2171, MM1.S, U87; mm9 for the mouse cell lines
- Resulting matrices feed directly into train_specific.py without dimension or format errors in the Keras CNN

## Limitations

- Only two pre-trained embedding matrices are supplied — human (hg19) and mouse (mm9); other genome builds require training a custom dna2vec model
- Sequences must be provided in FASTA format and placed in directories organized by cell name or the script will not find them
- The embedding step is sequence-only; it does not incorporate ChIP-seq or other epigenomic signals

## Evidence
