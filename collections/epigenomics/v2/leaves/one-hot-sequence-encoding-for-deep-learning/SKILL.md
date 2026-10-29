---
name: one-hot-sequence-encoding-for-deep-learning
description: Use when sgRNA sequence data must be converted into fixed-width one-hot
  channel matrices (e.g. 4 sequence channels over the 40-nt target window) to serve
  as deep learning input for CRISPR efficiency models.
when_to_use_negative:
- The input is already a numeric feature table (e.g. precomputed per-sgRNA features)
  rather than raw nucleotide sequences requiring channel encoding
- The target model is not a convolutional architecture and does not consume image-like
  channel matrices (e.g. gradient boosting or random forest baselines in the paper
  used different feature representations)
- Sequences vary in length or are not aligned to a fixed-width target window, since
  the encoding assumes a constant 40-nt target-site representation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Framework whose CNN consumes the one-hot encoded 4 x 40 sequence (and 11 x
    40 sequence+epigenetic) matrices as input
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for the deep learning models that take the one-hot encoded matrices
    as input
- name: pybedtools 0.9.0
  role: Used in the custom Python scripts that prepare/extract target-site coordinates
    and sequence windows for encoding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:79c52a4abc1ea7cbf31b2a4727e7054eb50d03e8f46fa8327f28e1fa9b4c808a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python
  - TensorFlow 2.4.1
  - pybedtools 0.9.0
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# one-hot-sequence-encoding-for-deep-learning

## Summary

Encodes sgRNA sequences (and optional epigenetic tracks) into fixed-width, image-like one-hot channel matrices suitable as CNN input for EpiCas-DL-style sgRNA editing-efficiency prediction, following the image-like coding scheme used in the EpiCas-DL paper.

## When to use

Use when sgRNA sequence data must be converted into fixed-width one-hot channel matrices (e.g. 4 sequence channels over the 40-nt target window) to serve as deep learning input for CRISPR efficiency models.

## When NOT to use

- The input is already a numeric feature table (e.g. precomputed per-sgRNA features) rather than raw nucleotide sequences requiring channel encoding
- The target model is not a convolutional architecture and does not consume image-like channel matrices (e.g. gradient boosting or random forest baselines in the paper used different feature representations)
- Sequences vary in length or are not aligned to a fixed-width target window, since the encoding assumes a constant 40-nt target-site representation

## Inputs

- sgRNA target-site sequences (40-nt windows, e.g. 'GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC')
- Epigenetic feature tracks per target site: nucleosome positioning/TSS distance, chromatin accessibility (ATACdb), DNA methylation, gene expression
- Preprocessed CRISPRoff/CRISPRi/CRISPRa screening datasets with activity-score-normalized phenotype scores

## Outputs

- One-hot encoded 4 x 40 sequence matrices (image-like channel representation)
- Stacked 11 x 40 matrices combining 4 sequence channels with 7 epigenetic channels (4 nucleosome/TSS, 1 accessibility, 1 methylation, 1 expression)
- Encoded matrices ready for TensorFlow CNN training and channel-subset ablation

## How to apply

Formulate each sgRNA sequence into an image-like one-hot coding scheme over the target-site window, matching EpiCas-DL's representation where sequences become 4 x 40 sequence channels that can be stacked with epigenetic channels (nucleosome-positioning/TSS distance, chromatin accessibility from ATACdb, DNA methylation, and gene expression) to form 11 x 40 matrices. Min-max normalize all epigenetic feature channels before model input. Use the encoded matrices as input to a CNN (five convolution filter sizes, 30 filters each of 1-5 nt) trained with TensorFlow; the same encoding supports ablation variants by selecting channel subsets (sequence-only vs. sequence plus single epigenetic features). The encoded data feeds a 90%/10% train/test split with 5-fold cross-validation for hyperparameter tuning via GPyOpt Bayesian optimization.

## Related tools

- **EpiCas-DL** (Framework whose CNN consumes the one-hot encoded 4 x 40 sequence (and 11 x 40 sequence+epigenetic) matrices as input) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for the deep learning models that take the one-hot encoded matrices as input)
- **pybedtools 0.9.0** (Used in the custom Python scripts that prepare/extract target-site coordinates and sequence windows for encoding)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- Encoded matrices have fixed shape: 4 channels x 40 positions for sequence-only, or 11 x 40 when stacking all epigenetic channels
- Each one-hot sequence position sums to 1 across the 4 nucleotide channels (one-hot constraint holds at every position)
- All epigenetic channels are min-max normalized to [0, 1] before model input
- Downstream CNN trained on 90%/10% split with 5-fold cross-validation reproduces the expected performance ordering: all-features EpiCas-DL > sequence-plus-single-epigenetic-feature models > sequence-only in ROC-AUC and Spearman correlation

## Limitations

- Incomplete epigenetic feature information limited EpiCas-DL performance, so encoding quality depends on availability of methylation, accessibility, expression, and nucleosome-positioning tracks
- The fixed 40-nt window means the encoding cannot represent variable-length sequence contexts
- The framework was developed on CRISPRoff/CRISPRi/CRISPRa screening data mainly from HEK293T and K562 cells, which may limit generalization of models built on these encodings

## Evidence

- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [readme] README on the four combined epigenetic features: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
- [other] The model input representation stacking sequence and epigenetic channels: [evidence span withheld — anchor status: not_found]
