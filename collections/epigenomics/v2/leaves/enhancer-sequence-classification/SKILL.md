---
name: enhancer-sequence-classification
description: Use when labeled enhancer FASTA sequences (e.g. 'data/train/${cell_line}.fasta'
  with '${cell_line}_y_train.txt' labels) must be classified per cell line, or generic/enhancer-promoter
  benchmarks must be reproduced.
when_to_use_negative:
- Input is already an extracted feature table — dna2vec_code.py and motif_find.py
  expect raw FASTA sequences, so skip feature extraction only when 'feature' outputs
  already exist
- The goal is transcription-factor motif interpretation rather than classification
  — use SHAP.py for exploring important motifs instead of the prediction pipeline
- The target task is promoter or other non-enhancer genomic element classification
  outside the enhancer/enhancer-promoter scope of this framework
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: Enhancer-MDLF
  role: The multi-input deep learning framework being applied — compiles dna2vec and
    motif feature inputs into a classifier trained with focal loss and a five-fold
    iterative training strategy
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec
  role: Provides the pre-trained DNA k-mer vector representations used by dna2vec_code.py
    to encode variable-length k-mers
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow
  role: Backend (version 2.2.0) used to implement network.py and run main.py training/prediction
- name: scikit-learn
  role: Computes evaluation metrics (BACC, MCC, Sn, ACC) on independent test sets
- name: Enhancer-IF
  role: Baseline machine-learning predictor against which Enhancer-MDLF test metrics
    are compared
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:b2dfe32b81290ca9d134ba646e48e228274f3be4c8f204758e6f024014ee12f2
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
  - Enhancer-IF
  - Enhancer-MDLF
  - Python
  - TensorFlow
  - dna2vec
  - numpy
  - pip
  - scikit-learn
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# enhancer-sequence-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and evaluate Enhancer-MDLF, a multi-input deep learning framework that fuses dna2vec word-vector features with HOCOMOCO v11 motif-frequency features to classify DNA sequences as cell-specific enhancers, with transfer learning for cross-cell-line prediction.

## When to use

Use when labeled enhancer FASTA sequences (e.g. 'data/train/${cell_line}.fasta' with '${cell_line}_y_train.txt' labels) must be classified per cell line, or generic/enhancer-promoter benchmarks must be reproduced.

## When NOT to use

- Input is already an extracted feature table — dna2vec_code.py and motif_find.py expect raw FASTA sequences, so skip feature extraction only when 'feature' outputs already exist
- The goal is transcription-factor motif interpretation rather than classification — use SHAP.py for exploring important motifs instead of the prediction pipeline
- The target task is promoter or other non-enhancer genomic element classification outside the enhancer/enhancer-promoter scope of this framework

## Inputs

- Enhancer training and independent-test FASTA files per cell line ('data/train/${cell_line}.fasta', 'data/test/${cell_line}.fasta')
- Label files per split ('${cell_line}_y_train.txt', '${cell_line}_y_test.txt')
- Enhancer-promoter sequences from the 'EPdata' folder (first half labeled 0, second half 1)
- Liu et al. iEnhancer-2L generic enhancer sequences from the 'generic data' folder
- Pre-trained dna2vec DNA vectors ('dna2vec' folder)
- HOCOMOCO Human v11 motif PWMs and p-value threshold scores ('motif' folder)

## Outputs

- Feature files (dna2vec-based and motif-based) written to the 'feature' directory
- Trained Enhancer-MDLF models per cell line (or 'model' folder pre-trained models)
- Per-cell-line test-set predictions and evaluation metrics (BACC and six additional metrics; MCC, Sn, ACC on the generic/EP benchmarks)

## How to apply

Organize sequences as 'data/train/${cell_line}.fasta' with '${cell_line}_y_train.txt' and 'data/test/${cell_line}.fasta' with '${cell_line}_y_test.txt' (EPdata files label the first half 0 and second half 1). In a Python 3.8 conda env with tensorflow 2.2.0, scikit-learn and numpy, extract dna2vec features via dna2vec_code.py using the pre-trained dna2vec vectors, and motif features via motif_find.py scanning sequences against the HOCOMOCO Human v11 PWMs and p-value thresholds; both write to the 'feature' directory. Then train the multi-input network (network.py via main.py) with focal loss under the five-fold iterative training strategy and predict on the independent test set, computing metrics with scikit-learn. Validation compares against reported values, e.g. MCC 0.6067, SN 0.84, ACC 0.8025 on the Liu et al. generic benchmark and the ~9.54% BACC improvement over Enhancer-IF on HMEC. Alternatively build and run the Docker image: docker build -t test . then docker run --rm -it -e PARAM1=NHLF test.

## Related tools

- **Enhancer-MDLF** (The multi-input deep learning framework being applied — compiles dna2vec and motif feature inputs into a classifier trained with focal loss and a five-fold iterative training strategy) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec** (Provides the pre-trained DNA k-mer vector representations used by dna2vec_code.py to encode variable-length k-mers) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow** (Backend (version 2.2.0) used to implement network.py and run main.py training/prediction)
- **scikit-learn** (Computes evaluation metrics (BACC, MCC, Sn, ACC) on independent test sets)
- **Enhancer-IF** (Baseline machine-learning predictor against which Enhancer-MDLF test metrics are compared)

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python main.py --cell_line NHLF
```

## Evaluation signals

- Feature extraction completes for both train and test splits of every cell line and outputs land in the 'feature' directory
- Independent-test metrics on the Liu et al. generic benchmark approximate MCC 0.6067, SN 0.84, ACC 0.8025 (Figure 5D, Supplementary Table S12)
- Enhancer-MDLF beats Enhancer-IF on all five metrics across all eight cell lines, including the ~9.54% BACC gain on HMEC
- Ablation check: the combined dna2vec+motif model improves all six metrics over dna2vec-only and motif-only variants (Figure 3, Supplementary Tables S2/S3)
- Enhancer-promoter test metrics match Supplementary Table S14 within reasonable variance

## Limitations

- Conventional feature encoding schemes and simplistic designs (as in Enhancer-IF) yield suboptimal cell-specific performance, motivating the multi-input design
- Cell specificity degrades cross-cell-line prediction, addressed via transfer learning rather than naive per-model reuse
- Dependencies are pinned (Python 3.8, pip 23.3.1, tensorflow 2.2.0); the README recommends a fresh conda env named 'test' to avoid package conflicts
- No changelog found for the repository, so version drift of data or code cannot be tracked from the README

## Evidence

- [results] On the Liu et al. generic enhancer dataset, Enhancer-MDLF achieves optimal performance on the independent test set with MCC of 0.6067: [evidence span withheld — anchor status: not_found]
- [other] Train Enhancer-MDLF with focal loss under the five-fold iterative training strategy and predict on the independent test sets: [evidence span withheld — anchor status: not_found]
