---
name: genome-build-correspondence-human-mouse
description: Use when encoding enhancer DNA sequences (SEs/TEs) into dna2vec feature
  vectors and you must decide which pre-trained embedding matrix (human hg19 vs mouse
  mm9) corresponds to each cell-line dataset.
when_to_use_negative:
- The enhancer coordinates or sequences are already provided as an encoded feature/matrix
  table rather than raw FASTA sequences
- The target organism or genome build is neither human hg19 nor mouse mm9 and no pre-trained
  embedding matrix is available (a new dna2vec model would need to be trained first)
- The downstream task uses ChIP-seq or other non-sequence features rather than dna2vec
  sequence embeddings
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: DeepSE
  role: Pipeline whose get_datavec.py encoding step consumes the species-matched FASTA
    sequences and embedding matrix
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Source of the pre-trained k-mer embedding matrices (hg19 and mm9) used to
    convert sequences into feature vectors
  repo: https://ngpatrick.com/dna2vec/
- name: Python
  role: Runtime (Python >=3.6) in which get_datavec.py is executed to encode each
    cell-line dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:bd0b4d6e736ff7f5daf05d5e46ac2e8daae530f06d9f42a4b315da853bf7ec75
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
  - Python
  - dna2vec
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# genome-build-correspondence-human-mouse

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches each enhancer dataset to the correct genome build so pre-trained dna2vec k-mer embedding matrices are applied consistently: human cell lines use hg19 and mouse cell lines use mm9 when encoding SE/TE sequences in the DeepSE pipeline.

## When to use

Use when encoding enhancer DNA sequences (SEs/TEs) into dna2vec feature vectors and you must decide which pre-trained embedding matrix (human hg19 vs mouse mm9) corresponds to each cell-line dataset.

## When NOT to use

- The enhancer coordinates or sequences are already provided as an encoded feature/matrix table rather than raw FASTA sequences
- The target organism or genome build is neither human hg19 nor mouse mm9 and no pre-trained embedding matrix is available (a new dna2vec model would need to be trained first)
- The downstream task uses ChIP-seq or other non-sequence features rather than dna2vec sequence embeddings

## Inputs

- SE and TE DNA sequence files in FASTA format, organized in directories by cell name
- Pre-trained dna2vec embedding matrices: human (hg19) and mouse (mm9)
- Cell-line dataset names from the DeepSE collection (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)

## Outputs

- Encoded k-mer feature-vector matrices written to './data/<cell_name>' for each dataset

## How to apply

Organize the FASTA files of SE and TE sequences by cell name for the nine datasets (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87). Assign the mouse mm9 pre-trained embedding matrix to the mouse-derived datasets (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell) and the human hg19 pre-trained embedding matrix to the human datasets (H2171, MM1.S, U87). Run the encoding script per cell-line/genome pair with 'python get_datavec.py cell_name genome' so the k-mer feature vectors are drawn from the species-matched embedding matrix; the encoded matrices are written to './data/<cell_name>'. The rationale is that k-mer feature vectors are genome-build specific, so a mouse sequence encoded with the hg19 matrix (or vice versa) would produce mismatched embeddings. If a different genome build is required, the dna2vec model can be retrained to obtain a custom embedding matrix.

## Related tools

- **DeepSE** (Pipeline whose get_datavec.py encoding step consumes the species-matched FASTA sequences and embedding matrix) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Source of the pre-trained k-mer embedding matrices (hg19 and mm9) used to convert sequences into feature vectors) — https://ngpatrick.com/dna2vec/
- **Python** (Runtime (Python >=3.6) in which get_datavec.py is executed to encode each cell-line dataset)

## Examples

```
python get_datavec.py mESC mm9
```

## Evaluation signals

- The encoding script completes without error for each cell-line/genome pair (e.g. 'python get_datavec.py mESC mm9')
- Expected encoded matrix files exist for every dataset under './data/<cell_name>' (e.g. './data/mESC' for the mESC dataset)
- Human cell lines (H2171, MM1.S, U87) were encoded with the hg19 matrix and mouse cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell) with the mm9 matrix
- Encoded matrices reproduce the uploaded DeepSE encoded cell data for the nine datasets when compared against the repository release

## Limitations

- Only two pre-trained embedding matrices are provided — human (version hg19) and mouse (mm9) — so other genome builds or species require training a custom dna2vec model
- The genome argument must match the species of the cell-line dataset; cross-species mismatches are not handled automatically
- Sequence files must already be in FASTA format and placed in directories named by cell name before encoding

## Evidence
