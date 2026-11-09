---
name: focal-loss-deep-learning-model-training
description: Use when training a deep learning enhancer predictor on imbalanced FASTA-encoded
  DNA sequence data (e.g., the eight cell lines in the Enhancer-IF benchmark with
  'data/train/${cell_line}.fasta' and '${cell_line}_y_train.txt' labels), or when
  reproducing Enhancer-MDLF vs Enhancer-IF comparisons.
when_to_use_negative:
- The task is generic enhancer or enhancer-promoter identification (use the 'EPdata'
  or 'generic data' workflows instead of the cell-specific benchmark pipeline)
- The input is already a feature matrix — this skill expects raw FASTA sequences from
  which dna2vec and motif features must first be extracted
- You only need motif interpretability (use SHAP.py / the 'force-plot' outputs) rather
  than model training and prediction
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Enhancer-MDLF
  role: The multi-input deep learning framework trained with focal loss on dna2vec
    and motif features to predict cell-specific enhancers
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow
  role: Deep learning backend (version 2.2.0) used to compile and run the Enhancer-MDLF
    network with focal loss
- name: scikit-learn
  role: Used for evaluation metric computation on the independent test sets
- name: dna2vec
  role: Provides pre-trained DNA vectors used by dna2vec_code.py to encode enhancer
    sequences into word-vector features
- name: Enhancer-IF
  role: Baseline integrated-ML enhancer predictor against which Enhancer-MDLF results
    are compared on the same benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:4a8faecb50df541ae445ca9f5aebec28e8fb379100110fd9ae2f091b69aae63d
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
  - pip
  - scikit-learn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# focal-loss-deep-learning-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains Enhancer-MDLF, a multi-input deep learning framework that combines dna2vec word-vector features and HOCOMOCO v11 motif-frequency features with a focal loss to predict cell-specific enhancers across eight human cell lines. The focal loss handles the class imbalance inherent to enhancer identification, and the model is trained per cell line and evaluated on independent test sets against Enhancer-IF.

## When to use

Use when training a deep learning enhancer predictor on imbalanced FASTA-encoded DNA sequence data (e.g., the eight cell lines in the Enhancer-IF benchmark with 'data/train/${cell_line}.fasta' and '${cell_line}_y_train.txt' labels), or when reproducing Enhancer-MDLF vs Enhancer-IF comparisons.

## When NOT to use

- The task is generic enhancer or enhancer-promoter identification (use the 'EPdata' or 'generic data' workflows instead of the cell-specific benchmark pipeline)
- The input is already a feature matrix — this skill expects raw FASTA sequences from which dna2vec and motif features must first be extracted
- You only need motif interpretability (use SHAP.py / the 'force-plot' outputs) rather than model training and prediction

## Inputs

- Training-set enhancer sequences per cell line ('data/train/${cell_line}.fasta')
- Training-set labels ('data/train/${cell_line}_y_train.txt')
- Independent test-set sequences and labels ('data/test/${cell_line}.fasta', '${cell_line}_y_test.txt')
- dna2vec features extracted via dna2vec_code.py (pre-trained dna2vec vectors)
- Motif features extracted via motif_find.py (HOCOMOCO Human v11 PWMs and p-value thresholds)

## Outputs

- Trained per-cell-line Enhancer-MDLF models (also pre-trained models on all cell lines in the 'model' folder)
- Test-set predictions for each cell line
- Five evaluation metrics (including BACC) per cell line for comparison against Enhancer-IF

## How to apply

First prepare the benchmark data in the repository layout (training FASTA and '_y_train.txt' label files, plus independent test FASTA and '_y_test.txt' files per cell line, e.g., NHLF). Set up a Python 3.8 conda environment (pip 23.3.1, tensorflow 2.2.0, scikit-learn, numpy) via 'pip install -r requirements.txt', then extract the two feature streams: dna2vec features with 'dna2vec_code.py' (using the pre-trained dna2vec vectors) and motif features with 'motif_find.py' (using HOCOMOCO Human v11 PWMs and p-value threshold scores); both write to the 'feature' directory. Train and predict per cell line with 'python main.py --cell_line ${cell_line}', which compiles the network defined in 'network.py' under the focal loss with the five-fold iterative training strategy and outputs predictions on the independent test set. Judge success by computing the five evaluation metrics (including BACC) on each independent test set and confirming Enhancer-MDLF outperforms Enhancer-IF on all metrics across all eight cell lines, including the reported 9.54% BACC improvement on HMEC.

## Related tools

- **Enhancer-MDLF** (The multi-input deep learning framework trained with focal loss on dna2vec and motif features to predict cell-specific enhancers) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow** (Deep learning backend (version 2.2.0) used to compile and run the Enhancer-MDLF network with focal loss)
- **scikit-learn** (Used for evaluation metric computation on the independent test sets)
- **dna2vec** (Provides pre-trained DNA vectors used by dna2vec_code.py to encode enhancer sequences into word-vector features)
- **Enhancer-IF** (Baseline integrated-ML enhancer predictor against which Enhancer-MDLF results are compared on the same benchmark)

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- Feature files for both dna2vec and motif streams exist in the 'feature' directory for each cell_line/set combination before main.py runs
- Enhancer-MDLF beats Enhancer-IF on all five evaluation metrics on the independent test sets of all eight cell lines
- The HMEC cell line reproduces approximately a 9.54% BACC improvement over Enhancer-IF
- Predictions are generated on the independent test set (e.g., 'data/test/NHLF.fasta' with 'NHLF_y_test.txt') rather than the training split
- Training completes under the five-fold iterative training strategy with focal loss without version conflicts in the Python 3.8 / tensorflow 2.2.0 environment

## Limitations

- Requires the exact dependency stack (Python 3.8, pip 23.3.1, tensorflow 2.2.0); the README recommends a fresh conda environment ('conda create -n test python=3.8') to avoid package conflicts
- Custom datasets must be reorganized into the exact repository layout ('data/train/${cell line name}.fasta', '${cell line name}_y_train.txt', etc.) before the pipeline can run
- Cross-cell line predictions suffer from enhancer cell specificity, which the paper mitigates via transfer learning rather than the per-cell-line focal-loss training alone

## Evidence
