---
name: transfer-learning-fine-tuning
description: Use when a trained multi-input deep learning model must be adapted to
  a new cell line with limited or divergent enhancer data, and the goal is to quantify
  which transfer strategy (transfer_1/2/3) counters enhancer specificity.
when_to_use_negative:
- The target cell line already has abundant labeled training data and the standard
  per-cell-line Enhancer-MDLF training path ('python main.py --cell_line ...') suffices
- Input is raw enhancer sequences without labels or without extracted features — feature
  extraction (dna2vec_code.py, motif_find.py) must be run first
- The task is generic enhancer (Liu et al. iEnhancer-2L data) or enhancer-promoter
  identification rather than cross-cell line transfer
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3307
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning framework whose pre-trained models (trained on all
    cell lines) are fine-tuned under the three transfer strategies
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow
  role: Deep learning backend (v2.2.0) for loading and fine-tuning the pre-trained
    Enhancer-MDLF models
- name: scikit-learn
  role: Used with numpy for computing predictive performance metrics of fine-tuned
    versus baseline models
- name: dna2vec
  role: Provides the pre-trained DNA vectors used to extract dna2vec-based features
    for each cell line before fine-tuning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:34038b6bbf6b3d7c1ddafed9cdaeaaf615a60f31453a35d1ca6fe26b116cf15f
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
  - numpy
  - scikit-learn
  - tensorflow
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# transfer-learning-fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes the pre-trained Enhancer-MDLF models (trained on all eight cell lines) on a target cell line to mitigate cross-cell line enhancer prediction challenges caused by enhancer specificity, comparing transfer_1, transfer_2, and transfer_3 strategies against random-weight retraining.

## When to use

Use when a trained multi-input deep learning model must be adapted to a new cell line with limited or divergent enhancer data, and the goal is to quantify which transfer strategy (transfer_1/2/3) best counters enhancer specificity.

## When NOT to use

- The target cell line already has abundant labeled training data and the standard per-cell-line Enhancer-MDLF training path ('python main.py --cell_line ...') suffices
- Input is raw enhancer sequences without labels or without extracted features — feature extraction (dna2vec_code.py, motif_find.py) must be run first
- The task is generic enhancer (Liu et al. iEnhancer-2L data) or enhancer-promoter identification rather than cross-cell line transfer

## Inputs

- Enhancer training set FASTA per cell line (e.g. 'data/train/${cell line}.fasta') with labels ('${cell line}_y_train.txt')
- Independent test set FASTA per cell line (e.g. 'data/test/${cell line}.fasta') with labels ('${cell line}_y_test.txt')
- Pre-trained Enhancer-MDLF models trained on all eight cell lines (from the 'model' folder)
- Pre-trained DNA vectors (dna2vec folder)
- Position weight matrices and p-value threshold scores from HOCOMOCO Human v11 (motif folder)

## Outputs

- Feature files for training and test sets saved in the 'feature' directory (dna2vec and motif features)
- Fine-tuned Enhancer-MDLF models under transfer_1, transfer_2, transfer_3 for each target cell line
- Baseline models retrained with random initial weights
- Comparison table of predictive performance metrics on independent test sets (cf. Supplementary Tables S8-S11, Figure 5C)

## How to apply

Load the enhancer training and independent test FASTA files with label files (e.g. 'data/train/NHLF.fasta' and 'NHLF_y_train.txt') for the target cell line, then extract dna2vec-based features with dna2vec_code.py using the pre-trained vectors in the 'dna2vec' folder and motif-based features with motif_find.py using PWMs and p-value thresholds from the HOCOMOCO Human v11 database. Load the pre-trained Enhancer-MDLF models from the 'model' folder that were trained on all cell lines, then run main.py to fine-tune under the three transfer-learning strategies (transfer_1, transfer_2, transfer_3) on the target cell line. As a baseline, retrain Enhancer-MDLF with random initial weights under otherwise identical settings. Evaluate all variants on the independent test set and assemble a comparison table (as in Supplementary Tables S8-S11); the validation criterion is that transfer_3 (pre-trained on all cell lines) ranks best, above transfer_2 (pre-training on multiple cell lines), transfer_1, and random initialization, consistent with Figure 5C.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning framework whose pre-trained models (trained on all cell lines) are fine-tuned under the three transfer strategies) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow** (Deep learning backend (v2.2.0) for loading and fine-tuning the pre-trained Enhancer-MDLF models)
- **scikit-learn** (Used with numpy for computing predictive performance metrics of fine-tuned versus baseline models)
- **dna2vec** (Provides the pre-trained DNA vectors used to extract dna2vec-based features for each cell line before fine-tuning)

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- Performance ranking across the eight cell lines places transfer_3 as the best strategy, above transfer_2, transfer_1, and random initialization (consistent with Figure 5C)
- transfer_2 (pre-training on multiple cell lines) yields more effective predictions in new cell lines than random initial weights
- Feature files for both training and test sets exist in the 'feature' directory after running dna2vec_code.py and motif_find.py
- Baseline random-initialization retraining uses otherwise identical settings to the transfer variants, so metric differences are attributable to transfer strategy
- Fine-tuned models load pre-trained weights from the 'model' folder trained on all cell lines rather than random initial weights

## Limitations

- The transfer-learning experiment requires pre-trained models trained on all eight cell lines; with only a single-cell-line pre-trained model the transfer_3 strategy is unavailable
- Performance is cell-line dependent because of enhancer specificity, which is precisely the challenge the skill targets
- Workflow requires matching HOCOMOCO Human v11 PWMs and p-value threshold scores for motif features; substituting other motif resources is untested in this context

## Evidence

- [results] Among the three strategies applied to Enhancer-MDLF, transfer_3 (pre-trained on all cell lines) exhibits optimal performance across the eight cell lines: [evidence span withheld — anchor status: not_found]
- [supplementary] Load the pre-trained Enhancer-MDLF models from the 'model' folder that were trained on all cell lines.: [evidence span withheld — anchor status: not_found]
- [supplementary] verify that the performance ranking places transfer_3 as the best strategy, above transfer_2, transfer_1, and random initialization, consistent with the reported Figure 5C results.: [evidence span withheld — anchor status: not_found]
