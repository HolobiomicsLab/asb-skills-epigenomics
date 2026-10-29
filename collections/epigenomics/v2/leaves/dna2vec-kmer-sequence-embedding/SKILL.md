---
name: dna2vec-kmer-sequence-embedding
description: Use when raw DNA sequences of enhancer regions (FASTA format) must be
  converted into fixed numeric feature vectors for a downstream classifier, and no
  ChIP-seq or experimental signal data is available.
when_to_use_negative:
- Input already consists of encoded feature matrices or numeric feature tables (skip
  straight to CNN training scripts)
- The classification task relies on experimental ChIP-seq/epigenomic signal tracks
  rather than DNA sequence only — this skill is designed for the sequence-only setting
- The genome version is neither hg19 nor mm9 and no external dna2vec training resource
  is available or desired
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: dna2vec
  role: Provides the pre-trained k-mer embedding matrices (hg19, mm9) used by get_datavec.py
    to convert TE/SE DNA sequences into feature vectors; can be retrained for custom
    genomes
  repo: https://ngpatrick.com/dna2vec/
- name: DeepSE
  role: Hosts the get_datavec.py encoding pipeline and the downstream two-layer Keras
    CNN that consumes the dna2vec-encoded matrices
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Python
  role: Runtime environment (>=3.6) in which get_datavec.py encodes sequences and
    dependencies are installed via requirements.txt with pip
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:7f978d352a64e7a7f48118533fc899e74528fd6f3eb4fff6355fcd64199e3ff1
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# dna2vec-kmer-sequence-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Represent DNA sequences as dense feature embeddings by encoding TE/SE sequences into k-mer feature vectors with pre-trained dna2vec embedding matrices, so that a sequence-based classifier (e.g. DeepSE's two-layer CNN) can distinguish super-enhancers from typical enhancers without ChIP-seq data.

## When to use

Use when raw DNA sequences of enhancer regions (FASTA format) must be converted into fixed numeric feature vectors for a downstream classifier, and no ChIP-seq or experimental signal data is available.

## When NOT to use

- Input already consists of encoded feature matrices or numeric feature tables (skip straight to CNN training scripts)
- The classification task relies on experimental ChIP-seq/epigenomic signal tracks rather than DNA sequence only — this skill is designed for the sequence-only setting
- The genome version is neither hg19 nor mm9 and no external dna2vec training resource is available or desired

## Inputs

- DNA sequence files in FASTA format for TEs and SEs, organized per cell line (e.g. mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)
- Pre-trained dna2vec k-mer embedding matrix for genome version hg19 (human) or mm9 (mouse)
- Cell line name and genome version identifiers passed as CLI arguments

## Outputs

- Encoded feature-vector matrices for the cell-line dataset, written to './data/<cell_name>'
- Optionally, a custom dna2vec embedding matrix trained on the user's own genome sequences

## How to apply

Prepare the TE/SE DNA sequences in FASTA format and place them in the working directory according to the cell name. Run 'python get_datavec.py cell_name genome' to convert each sequence into feature vectors using pre-trained k-mer feature vectors, choosing the genome matching the data's species: human (hg19) or mouse (mm9) pre-trained embedding matrices are supplied in the DeepSE repository. The encoded matrices are written to './data/<cell_name>' and consumed directly by the Keras two-layer CNN training scripts (train_specific.py, train_integrative1/2/3.py). If neither hg19 nor mm9 fits, train a species-specific dna2vec embedding at https://ngpatrick.com/dna2vec/ before encoding. Rationale: dna2vec embeddings capture k-mer-level sequence semantics that let a CNN discriminate SEs from TEs using DNA sequence alone.

## Related tools

- **dna2vec** (Provides the pre-trained k-mer embedding matrices (hg19, mm9) used by get_datavec.py to convert TE/SE DNA sequences into feature vectors; can be retrained for custom genomes) — https://ngpatrick.com/dna2vec/
- **DeepSE** (Hosts the get_datavec.py encoding pipeline and the downstream two-layer Keras CNN that consumes the dna2vec-encoded matrices) — https://github.com/QiaoyingJi/DeepSE
- **Python** (Runtime environment (>=3.6) in which get_datavec.py encodes sequences and dependencies are installed via requirements.txt with pip)

## Examples

```
python get_datavec.py mESC mm9
```

## Evaluation signals

- Running 'python get_datavec.py mESC mm9' produces encoded matrices under './data/mESC' as documented
- Encoded matrices have consistent shape across the cell-line dataset so they can be fed to train_specific.py without reshaping errors
- Downstream DeepSE training on the encoded data reproduces the paper's outcome: DeepSE outperforming all state-of-the-art methods for distinguishing SEs from TEs using sequence only
- The embedding matrix species matches the dataset genome (hg19 for human cell lines like H2171/MM1.S/U87; mm9 for mouse lines like mESC/myotube)
- FASTA input is well-formed and sequences are correctly labeled as TE or SE before encoding

## Limitations

- Only hg19 and mm9 pre-trained embedding matrices are provided; other genomes require training a custom dna2vec embedding externally
- Sequences must be supplied in FASTA format and placed in the directory according to the cell name, or the encoding step fails
- Requires Python >= 3.6 and installation of dependencies from requirements.txt before encoding
- The pipeline was validated on a fixed set of cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87); generalization to new cell types rests on the finding that cell-type specific SEs may share hidden sequence patterns

## Evidence

- [readme] get_datavec.py encoding command and output location: [evidence span withheld — anchor status: not_found]
