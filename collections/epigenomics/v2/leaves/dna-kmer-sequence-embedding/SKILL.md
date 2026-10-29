---
name: dna-kmer-sequence-embedding
description: Use when raw enhancer DNA sequences (FASTA) must be converted into fixed
  feature vectors/matrices before feeding a CNN classifier such as DeepSE, and pre-trained
  k-mer embeddings exist for the genome.
when_to_use_negative:
- The sequences are already encoded — the DeepSE repository ships all encoded cell
  data (mESC_constituent through U87), so re-running get_datavec.py is unnecessary
- No pre-trained dna2vec embedding matrix exists for the target genome and training
  a custom one is out of scope for the immediate analysis
- The task requires ChIP-seq or other epigenomic signals rather than DNA-sequence-only
  features, which is outside DeepSE's scope
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: DeepSE
  role: Pipeline that consumes the encoded k-mer feature matrices; get_datavec.py
    performs the DNA-sequence-to-feature-vector conversion step
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Source of the pre-trained k-mer embedding matrices (hg19, mm9) used to vectorize
    SE/TE sequences; can be retrained for custom genomes
  repo: https://ngpatrick.com/dna2vec/
- name: Python
  role: Runtime environment (>=3.6) in which get_datavec.py is executed
- name: Keras
  role: Framework in which the downstream two-layer CNN (model.py) consumes the encoded
    matrices
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:a3df6a3e8aad27759b2c63c1f236ea8020a9989d1f846bde92debb4b4a60cf66
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
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# dna-kmer-sequence-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode super-enhancer (SE) and typical-enhancer (TE) DNA sequences in FASTA format into numerical k-mer feature matrices using pre-trained dna2vec embeddings (human hg19 and mouse mm9), producing the vectorized inputs required by the DeepSE two-layer CNN.

## When to use

Use when raw enhancer DNA sequences (FASTA) must be converted into fixed feature vectors/matrices before feeding a CNN classifier such as DeepSE, and pre-trained k-mer embeddings exist for the genome.

## When NOT to use

- The sequences are already encoded — the DeepSE repository ships all encoded cell data (mESC_constituent through U87), so re-running get_datavec.py is unnecessary
- No pre-trained dna2vec embedding matrix exists for the target genome and training a custom one is out of scope for the immediate analysis
- The task requires ChIP-seq or other epigenomic signals rather than DNA-sequence-only features, which is outside DeepSE's scope

## Inputs

- SE and TE DNA sequence files in FASTA format, organized in directories by cell line name (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)
- Pre-trained dna2vec k-mer embedding matrix for the relevant genome (human hg19 or mouse mm9)

## Outputs

- Encoded k-mer feature matrices for each cell line dataset, written to './data/<cell_name>' (e.g. './data/mESC')

## How to apply

Prepare the SE/TE DNA sequence files in FASTA format and organize them in directories named by cell line (e.g. mESC, H2171, U87). Select the matching pre-trained dna2vec embedding matrix for the genome: human hg19 for human cell lines (H2171, MM1.S, U87) and mouse mm9 for mouse cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell); alternatively train a custom dna2vec model for a new genome. Run get_datavec.py with the cell_name and genome arguments (e.g. 'python get_datavec.py mESC mm9') so each sequence is converted into k-mer feature vectors via the pre-trained embedding matrix. Verify the encoded matrices are written to './data/<cell_name>' for every dataset before passing them to train_specific.py or the train_integrative*.py scripts.

## Related tools

- **DeepSE** (Pipeline that consumes the encoded k-mer feature matrices; get_datavec.py performs the DNA-sequence-to-feature-vector conversion step) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Source of the pre-trained k-mer embedding matrices (hg19, mm9) used to vectorize SE/TE sequences; can be retrained for custom genomes) — https://ngpatrick.com/dna2vec/
- **Python** (Runtime environment (>=3.6) in which get_datavec.py is executed)
- **Keras** (Framework in which the downstream two-layer CNN (model.py) consumes the encoded matrices)

## Examples

```
python get_datavec.py mESC mm9
```

## Evaluation signals

- For every encoded dataset, the expected matrix files exist in './data/<cell_name>' (e.g. './data/mESC')
- get_datavec.py completes without error for each cell line/genome pair
- The correct embedding matrix is used per species: hg19 for human cell lines (H2171, MM1.S, U87) and mm9 for mouse cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell)
- Encoded matrix dimensions are consistent across SE and TE samples so the downstream CNN (model.py) can accept them as training input

## Limitations

- Only two pre-trained embedding matrices are provided (human hg19 and mouse mm9); other genomes require training a custom dna2vec model
- Input sequences must be supplied in FASTA format and placed in directories named according to the cell name, otherwise get_datavec.py cannot locate them
- The encoding step requires a Python >=3.6 runtime with the requirements.txt modules installed

## Evidence

- [readme] get_datavec.py usage example: [evidence span withheld — anchor status: not_found]
