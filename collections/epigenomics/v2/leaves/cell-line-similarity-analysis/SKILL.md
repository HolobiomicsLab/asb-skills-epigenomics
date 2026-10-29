---
name: cell-line-similarity-analysis
description: Use when you need to quantify how well a cell-specific enhancer model
  trained on one human cell line transfers to the other seven cell lines' test sets,
  and to rank cell line similarity from cross-prediction BACC.
when_to_use_negative:
- When the goal is within-cell-line enhancer identification on each cell line's own
  independent test set rather than cross-cell line transferability
- When working on the generic enhancer dataset (Liu et al. iEnhancer-2L, 'generic
  data' folder) or the enhancer-promoter task ('EPdata' folder) instead of the eight
  cell-line enhancer sets
- When only a single cell line's training and test data are available, since no source-target
  pairs can be formed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning framework (main.py, TensorFlow 2.2.0) trained per
    cell line and evaluated on the other seven cell lines' test sets
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Enhancer-IF
  role: Baseline integrated-ML framework whose average balanced accuracy values are
    compared against Enhancer-MDLF's cross-cell line results
- name: dna2vec
  role: Provides the pre-trained DNA vectors used by dna2vec_code.py to extract k-mer
    embedding features for each cell line's sequences
- name: scikit-learn
  role: Computes balanced accuracy for the 56 cross-cell line source-target evaluations
- name: tensorflow
  role: Backend (version 2.2.0) for compiling and running the Enhancer-MDLF network
    during training and cross-cell line prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:dca0de08511642b716cd5efbb9222be6807dd8075e7fcf10bbe940e700d5f484
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cell-line-similarity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the transferability of cell-specific enhancer models by training Enhancer-MDLF on each of eight human cell lines and evaluating it on the other seven cell lines' independent test sets. The 56 source-target balanced accuracy (BACC) values expose which cell lines are mutually similar (e.g., HUVEC-K562, NHEK-HMEC) and whether Enhancer-MDLF beats Enhancer-IF on average.

## When to use

Use when you need to quantify how well a cell-specific enhancer model trained on one human cell line transfers to the other seven cell lines' test sets, and to rank cell line similarity from cross-prediction BACC.

## When NOT to use

- When the goal is within-cell-line enhancer identification on each cell line's own independent test set rather than cross-cell line transferability
- When working on the generic enhancer dataset (Liu et al. iEnhancer-2L, 'generic data' folder) or the enhancer-promoter task ('EPdata' folder) instead of the eight cell-line enhancer sets
- When only a single cell line's training and test data are available, since no source-target pairs can be formed

## Inputs

- Enhancer training set FASTA sequences and label files for eight cell lines (data/train/${cell_line}.fasta and ${cell_line}_y_train.txt)
- Independent test set FASTA sequences and label files for eight cell lines (data/test/${cell_line}.fasta and ${cell_line}_y_test.txt)
- Pre-trained dna2vec DNA vectors (folder 'dna2vec')
- HOCOMOCO Human v11 motif PWMs and p-value threshold scores (folder 'motif')
- Extracted dna2vec and motif feature files in the 'feature' directory
- Trained Enhancer-MDLF models per cell line (folder 'model')

## Outputs

- Balanced accuracy (BACC) for all 56 cell-line source-target pairs
- Per-source-cell-line average cross-prediction BACC (e.g., HEK293 average BACC of 86.18%)
- cross_cell_line_bacc.csv tabulating cross-cell line results and the relative similarity ordering of cell line pairs

## How to apply

Load the enhancer training and independent test FASTA sequences plus labels for the eight cell lines from the 'data' folder of the Enhancer-MDLF repository (data/train/${cell_line}.fasta, ${cell_line}_y_train.txt, data/test/${cell_line}.fasta, ${cell_line}_y_test.txt). For every cell line and both train/test splits, extract dna2vec features with dna2vec_code.py (using the pre-trained dna2vec vectors) and motif features with motif_find.py (HOCOMOCO Human v11 PWMs and p-value threshold scores), saving outputs to the 'feature' directory. Train one Enhancer-MDLF model per cell line with main.py (TensorFlow 2.2.0 multi-input deep network, Python 3.8), then evaluate each trained model on the independent test sets of the other seven cell lines, computing balanced accuracy with scikit-learn for all 56 source-target pairs. Average the cross-cell line BACC per source cell line and compare against the reported Enhancer-IF average values; similar pairs such as HUVEC-K562 and NHEK-HMEC should show mutually higher cross-prediction performance, reproducing Supplementary Tables S6/S7.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning framework (main.py, TensorFlow 2.2.0) trained per cell line and evaluated on the other seven cell lines' test sets) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Enhancer-IF** (Baseline integrated-ML framework whose average balanced accuracy values are compared against Enhancer-MDLF's cross-cell line results)
- **dna2vec** (Provides the pre-trained DNA vectors used by dna2vec_code.py to extract k-mer embedding features for each cell line's sequences)
- **scikit-learn** (Computes balanced accuracy for the 56 cross-cell line source-target evaluations)
- **tensorflow** (Backend (version 2.2.0) for compiling and running the Enhancer-MDLF network during training and cross-cell line prediction)

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- All 56 cell-line source-target pairs yield a non-missing balanced accuracy value (8 source models x 7 target test sets)
- The HEK29F source model's average BACC across the other seven cell lines reproduces the reported 86.18%
- Enhancer-MDLF achieves the best average BACC across all eight cell lines when compared to Enhancer-IF's reported averages
- HUVEC-K562 and NHEK-HMEC pairs show mutually higher cross-prediction BACC than unrelated cell line pairs, preserving the similarity ordering
- cross_cell_line_bacc.csv reproduces the patterns of Supplementary Tables S6/S7

## Limitations

- Cell line similarity is inferred only from cross-prediction performance, so it is conditional on the dna2vec + HOCOMOCO v11 motif feature space and the Enhancer-MDLF architecture
- Cross-cell line predictions are confounded by enhancer specificity, which is why the paper introduces transfer learning as a mitigation rather than assuming direct transfer works
- Feature extraction (dna2vec and motif_find.py) must be run for every train/test split and cell line, and the environment is pinned (Python 3.8, pip 23.3.1, tensorflow 2.2.0) to avoid dependency conflicts

## Evidence

- [readme] The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines.: "The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines."
- [results] In the cross-cell line evaluation, Enhancer-MDLF achieves the best average BACC across all eight cell lines; for example, the HEK293 model attains an average BACC of 86.18%: [evidence span withheld — anchor status: not_found]
- [other] Tabulate the results and verify that similar cell line pairs (e.g., HUVEC-K562 and NHEK-HMEC) show higher cross-prediction performance, reproducing the patterns in Supplementary Tables S6/S7.: [evidence span withheld — anchor status: not_found]
