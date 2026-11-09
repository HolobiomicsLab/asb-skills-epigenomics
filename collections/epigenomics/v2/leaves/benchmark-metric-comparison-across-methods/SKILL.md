---
name: benchmark-metric-comparison-across-methods
description: Use when two or more enhancer-prediction methods must be compared on
  a shared benchmark with independent test sets and per-cell-line metrics such as
  BACC, to verify a claimed improvement.
when_to_use_negative:
- Input sequences are not already split into training and independent test sets for
  the Enhancer-IF benchmark — random resampling would not support the head-to-head
  comparison on independent test data
- The goal is generic enhancer identification or enhancer-strength classification
  (Liu et al. iEnhancer-2L generic dataset) rather than cell-specific comparison across
  the eight cell lines
- Only one method is being evaluated with no published baseline metric values to compare
  against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3511
tools:
- name: Enhancer-MDLF
  role: Candidate method trained with focal loss under the five-fold iterative training
    strategy and evaluated on each independent test set
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Enhancer-IF
  role: Dedicated cell-specific enhancer predictor whose published results serve as
    the baseline for the head-to-head comparison
- name: dna2vec_code.py
  role: Extracts dna2vec-based features for the training and test splits feeding the
    compared model
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: motif_find.py
  role: Extracts motif-based features from HOCOMOCO Human v11 PWMs for the training
    and test splits
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: scikit-learn
  role: Used to compute the evaluation metrics on the independent test set predictions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:f9a12c401605198eafb0aea0798c2f59719dffe63e29413591e1b5fd06e59416
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
  - dna2vec_code.py
  - motif_find.py
  - numpy
  - pip
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# benchmark-metric-comparison-across-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a head-to-head benchmark in which a new predictor (Enhancer-MDLF) and a prior method (Enhancer-IF) are evaluated on the same independent test sets across eight human cell lines, tabulating five metrics to quantify performance differences (e.g. the 9.54% BACC gain on HMEC).

## When to use

Use when two or more enhancer-prediction methods must be compared on a shared benchmark with independent test sets and per-cell-line metrics such as BACC, to verify a claimed improvement.

## When NOT to use

- Input sequences are not already split into training and independent test sets for the Enhancer-IF benchmark — random resampling would not support the head-to-head comparison on independent test data
- The goal is generic enhancer identification or enhancer-strength classification (Liu et al. iEnhancer-2L generic dataset) rather than cell-specific comparison across the eight cell lines
- Only one method is being evaluated with no published baseline metric values to compare against

## Inputs

- Per-cell-line FASTA training and independent test enhancer sequences ('data/train/${cell_line}.fasta', 'data/test/${cell_line}.fasta')
- Per-cell-line label files ('${cell_line}_y_train.txt', '${cell_line}_y_test.txt')
- Pre-trained dna2vec DNA vectors (repository 'dna2vec' folder)
- HOCOMOCO Human v11 motif PWMs and p-value threshold scores (repository 'motif' folder)
- Published Enhancer-IF benchmark metric values for the eight cell lines

## Outputs

- Per-cell-line metric table (five evaluation metrics) for Enhancer-MDLF on independent test sets
- Side-by-side comparison table of Enhancer-MDLF vs Enhancer-IF metrics per cell line
- Quantified performance deltas, e.g. the 9.54% BACC improvement on the HMEC cell line

## How to apply

Prepare the Enhancer-IF benchmark data for each of the eight cell lines ('data/train/${cell_line}.fasta' with '${cell_line}_y_train.txt' and 'data/test/${cell_line}.fasta' with '${cell_line}_y_test.txt'), keeping training and independent test splits strictly separate. Set up Python 3.8 with tensorflow 2.2.0, scikit-learn and numpy via 'pip install -r requirements.txt', then extract both feature types per split: dna2vec-based features with dna2vec_code.py (using the pre-trained dna2vec vectors) and motif-based features with motif_find.py (HOCOMOCO Human v11 PWMs and p-value thresholds), writing outputs to the 'feature' directory. Train Enhancer-MDLF with focal loss under the five-fold iterative training strategy and predict on each independent test set via 'python main.py --cell_line ${cell_line}'. Compute the five evaluation metrics per cell line on the independent test sets and tabulate them against the reported Enhancer-IF results, confirming Enhancer-MDLF outperforms Enhancer-IF on all metrics, including the ~9.54% BACC improvement on HMEC.

## Related tools

- **Enhancer-MDLF** (Candidate method trained with focal loss under the five-fold iterative training strategy and evaluated on each independent test set) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Enhancer-IF** (Dedicated cell-specific enhancer predictor whose published results serve as the baseline for the head-to-head comparison)
- **dna2vec_code.py** (Extracts dna2vec-based features for the training and test splits feeding the compared model) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **motif_find.py** (Extracts motif-based features from HOCOMOCO Human v11 PWMs for the training and test splits) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **scikit-learn** (Used to compute the evaluation metrics on the independent test set predictions)

## Examples

```
python dna2vec_code.py --input_file data/test/NHLF.fasta --cell_line NHLF --set test; python motif_find.py --input_file data/test/NHLF.fasta --cell_line NHLF --set test; python main.py --cell_line NHLF
```

## Evaluation signals

- Feature files for both dna2vec and motif inputs exist in the 'feature' directory for every cell line and both train and test sets before running main.py
- Metrics are computed only on the independent test sets ('${cell_line}_y_test.txt'), never on training data
- Enhancer-MDLF outperforms Enhancer-IF on all five evaluation metrics in all eight cell lines
- The HMEC cell line reproduces an approximately 9.54% BACC improvement over Enhancer-IF
- Environment matches the paper's setup: Python 3.8, pip 23.3.1, tensorflow 2.2.0, scikit-learn, numpy

## Limitations

- The comparison is restricted to the eight human cell lines of the Enhancer-IF benchmark; conclusions do not automatically extend to other cell types
- Baseline Enhancer-IF values are taken from previously reported results rather than re-run locally, so the comparison depends on the fidelity of published numbers
- Dependencies are pinned (tensorflow 2.2.0, Python 3.8); newer environments may require the provided Docker image to avoid conflicts
