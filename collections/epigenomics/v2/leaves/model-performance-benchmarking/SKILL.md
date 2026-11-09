---
name: model-performance-benchmarking
description: Use when comparing a newly trained cell-specific enhancer classifier
  against a published baseline (e.g. Enhancer-IF) across multiple cell lines, or when
  quantifying cross-cell-line transferability via per-pair balanced accuracy.
when_to_use_negative:
- The goal is generic (non cell-specific) enhancer identification on the Liu et al.
  iEnhancer-2L generic dataset or enhancer-promoter (EPdata) data rather than cross-cell-line
  benchmarking
- Only a single cell line's within-cell-line test performance is needed — use the
  standard per-cell-line prediction workflow (main.py) without cross-evaluation
- Input is already an extracted feature table rather than raw FASTA sequences requiring
  dna2vec and motif feature extraction
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3511
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning framework (main.py, network.py, TensorFlow 2.2.0)
    trained per cell line and evaluated cross-cell-line with balanced accuracy
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Enhancer-IF
  role: Baseline integrated machine learning framework whose reported average BACC
    values serve as the comparison target
- name: dna2vec
  role: Source of pre-trained DNA k-mer vectors used by dna2vec_code.py for feature
    extraction prior to benchmarking
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: scikit-learn
  role: Library used for computing balanced accuracy metrics during evaluation
- name: tensorflow
  role: Framework (version 2.2.0) in which the Enhancer-MDLF network is compiled and
    trained for each benchmark run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:68e603fc28f12ac48394efd160ac73041f8564affcef11e01e1014b4a51d67d2
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# model-performance-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-specific enhancer prediction model (Enhancer-MDLF) against a baseline (Enhancer-IF) by training on each of eight human cell lines, evaluating balanced accuracy on independent test sets across all 56 cross-cell-line source-target pairs, and averaging per source cell line.

## When to use

Use when comparing a newly trained cell-specific enhancer classifier against a published baseline (e.g. Enhancer-IF) across multiple cell lines, or when quantifying cross-cell-line transferability via per-pair balanced accuracy.

## When NOT to use

- The goal is generic (non cell-specific) enhancer identification on the Liu et al. iEnhancer-2L generic dataset or enhancer-promoter (EPdata) data rather than cross-cell-line benchmarking
- Only a single cell line's within-cell-line test performance is needed — use the standard per-cell-line prediction workflow (main.py) without cross-evaluation
- Input is already an extracted feature table rather than raw FASTA sequences requiring dna2vec and motif feature extraction

## Inputs

- Enhancer training set FASTA sequences and labels per cell line (data/train/${cell_line}.fasta, ${cell_line}_y_train.txt)
- Independent test set FASTA sequences and labels per cell line (data/test/${cell_line}.fasta, ${cell_line}_y_test.txt)
- Pre-trained dna2vec DNA vectors
- HOCOMOCO Human v11 motif PWMs and p-value threshold scores

## Outputs

- dna2vec and motif feature files per cell line and split in the 'feature' directory
- Balanced accuracy per source-target cell-line pair (56 pairs for eight cell lines)
- Per-source-cell-line average BACC comparison table against Enhancer-IF (cross_cell_line_bacc.csv)

## How to apply

Load the enhancer training and independent test FASTA sequences with labels for the eight cell lines from the repository's 'data' folder (data/train/${cell line}.fasta, data/test/${cell line}.fasta plus _y_train.txt/_y_test.txt label files). For each cell line and each train/test split, extract dna2vec-based features (dna2vec_code.py with pre-trained dna2vec vectors) and motif features (motif_find.py using HOCOMOCO Human v11 PWMs and p-value threshold scores), saving outputs in the 'feature' directory. Train a TensorFlow 2.2.0 multi-input deep network (main.py) per cell line, then evaluate each trained model on the independent test sets of the other seven cell lines, computing balanced accuracy (BACC) for all 56 source-target pairs. Average cross-cell-line BACC per source cell line and compare against reported Enhancer-IF average BACC values, checking that similar cell line pairs (HUVEC-K562, NHEK-HMEC) show higher cross-prediction performance as in Supplementary Tables S6/S7.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning framework (main.py, network.py, TensorFlow 2.2.0) trained per cell line and evaluated cross-cell-line with balanced accuracy) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Enhancer-IF** (Baseline integrated machine learning framework whose reported average BACC values serve as the comparison target)
- **dna2vec** (Source of pre-trained DNA k-mer vectors used by dna2vec_code.py for feature extraction prior to benchmarking) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **scikit-learn** (Library used for computing balanced accuracy metrics during evaluation)
- **tensorflow** (Framework (version 2.2.0) in which the Enhancer-MDLF network is compiled and trained for each benchmark run)

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python main.py --cell_line NHLF
```

## Evaluation signals

- cross_cell_line_bacc.csv must reproduce the reported result that Enhancer-MDLF outperforms Enhancer-IF across the eight cell lines
- The HEK293-trained model should attain an average BACC of 86.18% when predicting the other seven cell lines
- Cross-prediction BACC for mutual pairs HUVEC-K562 and NHEK-HMEC must be higher than for dissimilar cell line pairs, preserving the relative similarity ordering of Supplementary Tables S6/S7
- Feature extraction must produce both dna2vec and motif feature files for every cell line and split (train/test) in the 'feature' directory before training can run
- All 8x7 = 56 source-target cell-line pairs must be evaluated with balanced accuracy and averaged per source cell line

## Limitations

- Benchmarks are specific to the eight cell lines and data organization provided in the repository; custom datasets must be reformatted into the same train/test FASTA plus label file structure
- Docker-based runs are demonstrated only for a single cell line (NHLF prediction results), so the containerized path does not directly cover the full 56-pair cross-evaluation
- Cross-cell-line transferability depends on cell-specific enhancer patterns; dissimilar cell line pairs are expected to show lower BACC, so absolute values are not comparable across pairings

## Evidence
