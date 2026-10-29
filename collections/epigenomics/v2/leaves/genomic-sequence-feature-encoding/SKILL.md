---
name: genomic-sequence-feature-encoding
description: Use when raw enhancer FASTA sequences (e.g. data/train/NHLF.fasta) must
  be converted into fixed-dimensional numeric vectors before training or predicting
  with a deep learning model such as Enhancer-MDLF.
when_to_use_negative:
- Input is already a feature table or numeric matrix rather than raw FASTA sequences
  — encoding would be redundant
- The target model expects other encodings (e.g. one-hot sequence tensors) instead
  of dna2vec k-mer embeddings
- Sequences are not DNA enhancer/promoter sequences in the project's FASTA layout,
  so the k-mer lookup vocabulary from the pre-trained dna2vec vectors does not apply
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: dna2vec
  role: Source of the pre-trained DNA vector representations looked up for each 3-mer
    during encoding
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: numpy
  role: Array operations for concatenating per-k-mer embeddings and adaptive pooling
    to the fixed 10,000-dim vector
- name: Enhancer-MDLF
  role: Downstream multi-input deep learning model that consumes the 10,000-dim dna2vec
    features together with motif features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec_code.py
  role: Script that performs the dna2vec feature extraction with --input_file, --cell_line,
    and --set arguments
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:6d7731f220e96a270d244be98f2ea20adcc0c55368120e490258f045029f7ad2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  repo_url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  tools:
  - Enhancer-MDLF
  - Python
  - dna2vec
  - dna2vec_code.py
  - numpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# genomic-sequence-feature-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes DNA enhancer sequences into fixed-length numeric feature vectors for the Enhancer-MDLF multi-input deep learning framework, using pre-trained dna2vec k-mer embeddings (k=3, 100-dim per k-mer) pooled to a 10,000-dimensional input, complemented by HOCOMOCO motif-frequency features.

## When to use

Use when raw enhancer FASTA sequences (e.g. data/train/NHLF.fasta) must be converted into fixed-dimensional numeric vectors before training or predicting with a deep learning model such as Enhancer-MDLF.

## When NOT to use

- Input is already a feature table or numeric matrix rather than raw FASTA sequences — encoding would be redundant
- The target model expects other encodings (e.g. one-hot sequence tensors) instead of dna2vec k-mer embeddings
- Sequences are not DNA enhancer/promoter sequences in the project's FASTA layout, so the k-mer lookup vocabulary from the pre-trained dna2vec vectors does not apply

## Inputs

- Enhancer FASTA sequences per cell line and set (e.g. 'data/train/NHLF.fasta', 'data/test/NHLF.fasta')
- Pre-trained dna2vec k-mer vectors from the 'dna2vec' folder
- CLI arguments: --input_file, --cell_line, --set (train or test)

## Outputs

- dna2vec feature files saved in the 'feature' directory, one fixed 10,000-dimensional vector per enhancer sequence
- Companion motif-based feature files from motif_find.py, also saved in the 'feature' directory

## How to apply

Load the enhancer FASTA file for the chosen cell line and set (e.g. 'data/train/NHLF.fasta') following the data-folder naming convention, then load the pre-trained DNA vectors shipped in the 'dna2vec' folder (vectors from Ng's dna2vec). For each sequence, slide a k-mer window with k=3 and stride 1, look up each 3-mer's pre-trained 100-dim embedding, and concatenate the embeddings along the sequence. Apply adaptive pooling over the concatenated per-k-mer embedding sequence so every enhancer, regardless of length, yields one fixed 10,000-dimensional feature vector, using numpy for array operations. Run the extraction separately for train and test sets via dna2vec_code.py with --input_file, --cell_line, and --set arguments, saving outputs to the 'feature' directory; in parallel, motif_find.py derives motif-frequency features from the HOCOMOCO Human v11 PWMs for the model's second input branch.

## Related tools

- **dna2vec** (Source of the pre-trained DNA vector representations looked up for each 3-mer during encoding) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **numpy** (Array operations for concatenating per-k-mer embeddings and adaptive pooling to the fixed 10,000-dim vector)
- **Enhancer-MDLF** (Downstream multi-input deep learning model that consumes the 10,000-dim dna2vec features together with motif features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec_code.py** (Script that performs the dna2vec feature extraction with --input_file, --cell_line, and --set arguments) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- Output feature files exist in the 'feature' directory after running dna2vec_code.py for both train and test sets
- Each sequence yields a feature vector of the fixed 10,000-dimensional input size expected by the Enhancer-MDLF network
- Feature extraction was run for matching cell_line and set values (e.g. NHLF train and NHLF test) so training and prediction use consistent encodings
- Downstream main.py --cell_line run trains and predicts without dimensionality errors, confirming feature shape compatibility
- Every 3-mer looked up exists in the pre-trained dna2vec vector file (no missing-key failures) since k is fixed at 3

## Limitations

- Embedding dimensionality and fixed 10,000-dim pooled length are tailored to Enhancer-MDLF's network; other models may need different sizes
- Only k=3-mers with stride 1 are covered by this encoding; k-mer sizes outside the pre-trained dna2vec vocabulary cannot be looked up
- Docker runs pass the cell line via PARAM1 environment variable (e.g. docker run --rm -it -e PARAM1=NHLF test), which must match the trained model's cell line

## Evidence
