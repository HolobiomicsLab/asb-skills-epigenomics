---
name: cross-cell-line-enhancer-prediction
description: Use when FASTA enhancer sequences with binary labels for specific human
  cell lines (e.g. NHLF, HEK293, HUVEC, K562, NHEK, HMEC) must be classified, or when
  models trained on one cell line must be evaluated on or transferred to other cell
  lines' test sets.
when_to_use_negative:
- Input is already an extracted feature table rather than raw enhancer FASTA sequences
  — dna2vec/motif extraction is unnecessary.
- The task is generic (non-cell-specific) enhancer classification on the Liu et al.
  iEnhancer-2L dataset or enhancer-promoter data (EPdata folder), which are separate
  experiment tracks rather than the eight cell-line-specific datasets.
- The goal is interpretation of which TFBS motifs matter — SHAP.py / 'force-plot'
  outputs are the appropriate skill, not cross-cell-line prediction.
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning framework (main.py, network.py, TensorFlow 2.2.0)
    trained per cell line and fine-tuned for transfer
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec
  role: Pre-trained DNA vectors used by dna2vec_code.py to extract k-mer word-vector
    features from enhancer FASTA sequences
- name: tensorflow
  role: Deep learning backend (2.2.0) for compiling and running Enhancer-MDLF
- name: scikit-learn
  role: Metric computation (e.g. balanced accuracy) on independent test sets
- name: Enhancer-IF
  role: Baseline method whose average BACC is compared against Enhancer-MDLF in cross-cell-line
    evaluation
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:00c2c4e794dffc7c1372d20dad085c60f92a064250f3ef93f62d7352797bfb64
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
  - dna2vec
  - numpy
  - scikit-learn
  - tensorflow
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# cross-cell-line-enhancer-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict cell-specific enhancers across multiple human cell lines using Enhancer-MDLF, a TensorFlow multi-input deep network combining dna2vec word-vector features and HOCOMOCO v11 motif features, and evaluate/transfer these models across cell lines (including transfer_1/2/3 strategies).

## When to use

Use when FASTA enhancer sequences with binary labels for specific human cell lines (e.g. NHLF, HEK293, HUVEC, K562, NHEK, HMEC) must be classified, or when models trained on one cell line must be evaluated on or transferred to other cell lines' test sets.

## When NOT to use

- Input is already an extracted feature table rather than raw enhancer FASTA sequences — dna2vec/motif extraction is unnecessary.
- The task is generic (non-cell-specific) enhancer classification on the Liu et al. iEnhancer-2L dataset or enhancer-promoter data (EPdata folder), which are separate experiment tracks rather than the eight cell-line-specific datasets.
- The goal is interpretation of which TFBS motifs matter — SHAP.py / 'force-plot' outputs are the appropriate skill, not cross-cell-line prediction.

## Inputs

- Training set FASTA per cell line: 'data/train/${cell line name}.fasta'
- Training labels: 'data/train/${cell line name}_y_train.txt'
- Independent test set FASTA: 'data/test/${cell line name}.fasta'
- Test labels: 'data/test/${cell line name}_y_test.txt'
- Pre-trained dna2vec DNA vectors ('dna2vec' folder)
- HOCOMOCO Human v11 PWMs and p-value threshold scores ('motif' folder)
- Pre-trained Enhancer-MDLF models ('model' folder) for transfer learning

## Outputs

- dna2vec and motif feature files saved in the 'feature' directory
- Trained Enhancer-MDLF model and per-cell-line test predictions
- Cross-cell-line balanced accuracy (BACC) for 56 cell-line source-target pairs (e.g. cross_cell_line_bacc.csv)
- Transfer-learning comparison table (cf. Supplementary Tables S8-S11, Figure 5C)

## How to apply

Organize training/test data as 'data/train/${cell line}.fasta' with '${cell line}_y_train.txt' label files and 'data/test/${cell line}.fasta' with '${cell line}_y_test.txt'. Extract dna2vec-based features (dna2vec_code.py using the pre-trained DNA vectors in the 'dna2vec' folder) and motif-based features (motif_find.py using HOCOMOCO Human v11 PWMs and p-value threshold scores from the 'motif' folder), saving features to the 'feature' directory. Train the multi-input deep network with 'python main.py --cell_line ${cell_line}' (network.py defines the architecture; TensorFlow 2.2.0, Python 3.8). For cross-cell-line transferability, evaluate each trained model on the independent test sets of the other seven cell lines, computing balanced accuracy (BACC) for all 56 source-target pairs; when enhancer specificity degrades transfer, fine-tune the pre-trained models in the 'model' folder under the three transfer-learning strategies, with transfer_3 (pre-trained on all cell lines) expected to perform best, above transfer_2, transfer_1, and random initialization.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning framework (main.py, network.py, TensorFlow 2.2.0) trained per cell line and fine-tuned for transfer) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec** (Pre-trained DNA vectors used by dna2vec_code.py to extract k-mer word-vector features from enhancer FASTA sequences)
- **tensorflow** (Deep learning backend (2.2.0) for compiling and running Enhancer-MDLF)
- **scikit-learn** (Metric computation (e.g. balanced accuracy) on independent test sets)
- **Enhancer-IF** (Baseline method whose average BACC is compared against Enhancer-MDLF in cross-cell-line evaluation)

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python main.py --cell_line NHLF
```

## Evaluation signals

- Feature extraction outputs exist in the 'feature' directory for both train and test sets of every cell line, matching the --cell_line and --set arguments supplied.
- Cross-cell-line evaluation covers all 56 (8x7) source-target cell-line pairs with balanced accuracy computed on the independent test sets.
- Enhancer-MDLF attains the best average BACC across all eight cell lines vs Enhancer-IF; e.g. the HEK293 model reaches an average BACC of 86.18% predicting other cell lines.
- Similar cell-line pairs (mutual HUVEC-K562 and NHEK-HMEC) show higher cross-prediction performance, reproducing the ordering in Supplementary Tables S6/S7.
- Transfer-learning ranking places transfer_3 > transfer_2 > transfer_1 > random initialization, consistent with Figure 5C.

## Limitations

- Cross-cell-line predictions are degraded by enhancer specificity, motivating but also constraining direct transfer without pre-training.
- Dependencies are pinned and tested on Python 3.8, pip 23.3.1, and TensorFlow 2.2.0; other environments may conflict (requirements.txt / conda env 'test' recommended).
- Custom datasets must be reorganized into the exact 'data/{train,test}/${cell line}' FASTA + label file format before the pipeline will run.
- Reported transfer-learning and cross-cell-line results are tied to the paper's eight cell lines and supplementary tables; unverified on other benchmarks.

## Evidence
