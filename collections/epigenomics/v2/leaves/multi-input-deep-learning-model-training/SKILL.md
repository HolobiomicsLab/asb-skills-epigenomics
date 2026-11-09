---
name: multi-input-deep-learning-model-training
description: Use when cell-line enhancer FASTA training/test sets with label files
  are available and pre-extracted dna2vec and motif features exist, and a combined
  multi-input model must be trained and evaluated on an independent test set.
when_to_use_negative:
- Input is already a feature table produced by dna2vec_code.py/motif_find.py only
  for a single feature type and the goal is a single-feature ablation baseline rather
  than the combined multi-input model
- Data are not enhancer sequences with training/test labels (e.g. non-FASTA genomics
  data or unlabelled sequences) — the framework requires organized training/test FASTA
  and label files
- Only cross-cell-line transfer with the provided pre-trained models is needed; use
  the pre-trained models in the 'model' folder instead of retraining
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Enhancer-MDLF
  role: The multi-input deep learning framework being trained; architecture defined
    in network.py, driven by main.py
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow 2.2.0
  role: Deep learning backend used to compile and train the multi-input network
- name: scikit-learn
  role: Computes evaluation metrics (BACC, MCC, Sn, ACC) on the independent test sets
- name: dna2vec
  role: Provides pre-trained DNA vectors from the 'dna2vec' folder used by dna2vec_code.py
    to build one input branch's features
- name: Python 3.8
  role: Runtime environment (recommended conda env 'test', pip install -r requirements.txt)
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:9b7b66a1d8079c2dc1cd90b7a800cab8e846776c648e0fbfc1ffa1376fec01b0
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
  - Python 3.8
  - TensorFlow
  - dna2vec
  - numpy
  - scikit-learn
  - tensorflow 2.2.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# multi-input-deep-learning-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train Enhancer-MDLF, a multi-input deep learning framework that fuses dna2vec word-vector features and HOCOMOCO motif-frequency features to predict cell-specific enhancers, using network.py and main.py in TensorFlow 2.2.0.

## When to use

Use when cell-line enhancer FASTA training/test sets with label files are available and pre-extracted dna2vec and motif features exist, and a combined multi-input model must be trained and evaluated on an independent test set.

## When NOT to use

- Input is already a feature table produced by dna2vec_code.py/motif_find.py only for a single feature type and the goal is a single-feature ablation baseline rather than the combined multi-input model
- Data are not enhancer sequences with training/test labels (e.g. non-FASTA genomics data or unlabelled sequences) — the framework requires organized training/test FASTA and label files
- Only cross-cell-line transfer with the provided pre-trained models is needed; use the pre-trained models in the 'model' folder instead of retraining

## Inputs

- Cell-line enhancer training and test FASTA files (data/train/${cell_line}.fasta, data/test/${cell_line}.fasta)
- Training and test label files (${cell_line}_y_train.txt, ${cell_line}_y_test.txt)
- Pre-extracted dna2vec-based features from dna2vec_code.py
- Pre-extracted motif-based features from motif_find.py (HOCOMOCO Human v11 PWMs and p-value thresholds)
- Optionally, generic enhancer ('generic data', Liu et al.) and enhancer-promoter ('EPdata') FASTA sets with half-0/half-1 label convention

## Outputs

- Trained Enhancer-MDLF models per cell line (model folder)
- Prediction results on the independent test set
- Performance metrics (BACC and six additional metrics; MCC/SN/ACC for generic and EP datasets) e.g. ablation_results.csv and generic_ep_evaluation_results.csv

## How to apply

After extracting dna2vec features (dna2vec_code.py with --input_file, --cell_line, --set) and motif features (motif_find.py scanning against HOCOMOCO Human v11 PWMs and p-value thresholds), both saved to the 'feature' directory, run main.py with a --cell_line argument to compile and train the multi-input architecture defined in network.py (TensorFlow 2.2.0, Python 3.8). The network has two input branches that fuse the dna2vec and motif feature representations into a single predictor. For each cell line, train on the training split from 'data/train/${cell_line}.fasta' with labels '${cell_line}_y_train.txt' and predict on the independent test set from 'data/test/'. Evaluate with scikit-learn metrics (including BACC and six additional reported metrics); the same workflow extends to the 'generic data' (Liu et al.) and 'EPdata' folders, where the first half of each file is label 0 and the second half label 1. As a decision point, compare against single-module variants (dna2vec-only, motif-only) to confirm the combined model's ablation advantage.

## Related tools

- **Enhancer-MDLF** (The multi-input deep learning framework being trained; architecture defined in network.py, driven by main.py) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow 2.2.0** (Deep learning backend used to compile and train the multi-input network)
- **scikit-learn** (Computes evaluation metrics (BACC, MCC, Sn, ACC) on the independent test sets)
- **dna2vec** (Provides pre-trained DNA vectors from the 'dna2vec' folder used by dna2vec_code.py to build one input branch's features)
- **Python 3.8** (Runtime environment (recommended conda env 'test', pip install -r requirements.txt))

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- All six reported metrics improve across the eight cell lines for the combined model versus dna2vec-only and motif-only variants (Figure 3, Supplementary Tables S2/S3 ablation pattern)
- On the Liu et al. generic enhancer dataset, independent-test metrics approximate MCC 0.6067, SN 0.84, ACC 0.8025 (Figure 5D, Supplementary Table S12)
- Feature files for both train and test splits of each cell line exist in the 'feature' directory before main.py is invoked
- Enhancer-MDLF outperforms the prior Enhancer-IF method across all eight human cell lines
- Enhancer-promoter test metrics match Supplementary Table S14 within reasonable variance

## Limitations

- Requires specific environment versions (Python 3.8, tensorflow 2.2.0, pip 23.3.1); dependency conflicts are avoided by a dedicated conda env or Docker image
- Training is cell-line specific: a separate model must be trained per cell line via --cell_line; cross-cell-line specificity issues are otherwise addressed by transfer learning
- Custom datasets must be reorganized into the exact README directory/label format ('data/train', 'data/test', half-0/half-1 label convention for EPdata)

## Evidence

- [other] Fusing the dna2vec and motif modules into the combined Enhancer-MDLF improves prediction: all six metrics exhibit improvement across the eight cell lines compared with the individually trained modules.: [evidence span withheld — anchor status: not_found]
