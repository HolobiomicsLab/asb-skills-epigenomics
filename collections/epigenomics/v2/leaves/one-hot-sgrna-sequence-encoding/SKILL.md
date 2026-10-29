---
name: one-hot-sgrna-sequence-encoding
description: Use when raw sgRNA nucleotide sequences must be converted into numeric
  tensors for the EpiCas-DL CNN (TensorFlow 2.4.1), either as 4×40 sequence-only matrices
  or stacked into 11×40 sequence-plus-epigenetics inputs.
when_to_use_negative:
- The input is already a one-hot encoded or feature-table representation — re-encoding
  would duplicate preprocessing
- Sequences do not fit the 40-bp (9+23+8) window layout around the PAM assumed by
  EpiCas-DL
- The goal is regression/classification on non-sequence tabular features (e.g. Gradient
  Boosting or Random Forest baselines can consume the features without image-style
  one-hot encoding)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Framework whose CNN defines the required one-hot encoding scheme (4 sequence
    channels, 40-bp window, 11×40 input matrix) and provides example.py for sequence-only
    or sequence-plus-epigenetics prediction
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend that consumes the one-hot encoded tensors when training/running the
    EpiCas-DL CNN models
- name: Python 3.6
  role: Language for the custom encoding/preprocessing scripts used with pybedtools
    in the EpiCas-DL pipeline
provenance:
  source_task_ids:
  - task_008
  - task_001
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:92471c4050d92c2e17d5179fbcbebc15f5fee5b2c536b22854500f76c6e92b0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - DeepSHAP
  - EpiCas-DL
  - GPyOpt
  - Python 3.6
  - TensorFlow 2.4.1
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# One-hot sgRNA sequence encoding

## Summary

Converts sgRNA nucleotide strings into the image-like one-hot tensor format required by EpiCas-DL: a 4-channel A/T/G/C encoding over a 40-bp window (9 bp upstream, 23 bp target, 8 bp downstream) that can be stacked with epigenetic channels into the 11×40 model input.

## When to use

Use when raw sgRNA nucleotide sequences must be converted into numeric tensors for the EpiCas-DL CNN (TensorFlow 2.4.1), either as 4×40 sequence-only matrices or stacked into 11×40 sequence-plus-epigenetics inputs.

## When NOT to use

- The input is already a one-hot encoded or feature-table representation — re-encoding would duplicate preprocessing
- Sequences do not fit the 40-bp (9+23+8) window layout around the PAM assumed by EpiCas-DL
- The goal is regression/classification on non-sequence tabular features (e.g. Gradient Boosting or Random Forest baselines can consume the features without image-style one-hot encoding)

## Inputs

- Raw sgRNA nucleotide sequence strings spanning a 40-bp window (9 bp upstream + 23 bp target + 8 bp downstream), e.g. GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC
- Optional epigenetic features aligned to the sgRNA locus (nucleosome positioning, ATAC-seq chromatin accessibility from ATACdb, DNA methylation, RNA expression) for min–max normalization and stacking into the 11×40 matrix

## Outputs

- One-hot encoded 4-channel (A/T/G/C) × 40-position binary matrix per sgRNA
- 11×40 feature tensor per sgRNA when epigenetic channels (4 nucleosome, 1 accessibility, 1 methylation, 1 expression) are stacked on top of the sequence channels
- Encoded input batch suitable for the EpiCas-DL CNN and subsequent DeepSHAP per-base attribution

## How to apply

Take each sgRNA's 40-bp context (9 bp upstream, 23 bp target, 8 bp downstream of the PAM) and one-hot encode it into four binary A/T/G/C channels per position, producing an image-like matrix. To build the full model input, stack these 4 sequence channels with min–max normalized epigenetic channels (4 nucleosome positioning, 1 chromatin accessibility, 1 DNA methylation, 1 gene expression) into an 11×40 tensor. When epigenetic features are unavailable, use the sequence-only path exposed by the EpiCas-DL repository's example.py with a *_seq model (e.g. CRISPRoff_seq); when coordinates, cell line and gene are available, supply chrom/start/end/pamCoord/transcript/gene/cell_line for a *_epi model. The rationale is that the EpiCas-DL CNN's multi-size convolutional layer (30 filters each at 1–5 nt widths) operates on this encoded matrix, so the encoding must preserve exact per-position base identity for downstream SHAP attribution at positions such as the PAM-proximal position 20.

## Related tools

- **EpiCas-DL** (Framework whose CNN defines the required one-hot encoding scheme (4 sequence channels, 40-bp window, 11×40 input matrix) and provides example.py for sequence-only or sequence-plus-epigenetics prediction) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend that consumes the one-hot encoded tensors when training/running the EpiCas-DL CNN models)
- **Python 3.6** (Language for the custom encoding/preprocessing scripts used with pybedtools in the EpiCas-DL pipeline)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- Each encoded sgRNA yields a 4×40 binary matrix with exactly one active channel per position
- Stacking with the 7 epigenetic channels (4 nucleosome positioning + 1 accessibility + 1 methylation + 1 expression) reproduces the 11×40 input tensor shape
- example.py runs without error on a 40-nt sequence and produces a prediction from the corresponding CRISPRoff_seq or CRISPRoff_epi model
- Encoded sgRNAs fed through the trained CNN with 10-fold cross-validation on a 90/10 split reach the expected performance (CRISPRoff AUC ≈ 0.879)
- DeepSHAP attributions computed on encoded inputs resolve per-position nucleotide contributions (e.g. guanine favored at PAM-proximal position 20/24 in the CRISPRoff model)

## Limitations

- The sequence-only encoding discards epigenetic context (methylation, chromatin accessibility, gene expression) that improves sgRNA activity prediction; the article notes performance was limited by incomplete epigenetic feature information
- Differences between CRISPRoff and CRISPRi systems can affect prediction performance when transferring models trained on one to datasets of the other
- Screening data are mainly from HEK293T and K562 cells, so encoded sgRNAs from other contexts carry overfitting and generalization risk

## Evidence

- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [readme] The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
- [intro] Our EpiCas-DL model competes favorably with the available state-of-the-art tools in independent validation cohorts and different cell types: "Our EpiCas-DL model competes favorably with the available state-of-the-art tools in independent validation cohorts and different cell types"
