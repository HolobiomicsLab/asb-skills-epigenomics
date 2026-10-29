---
name: promoter-enhancer-discrimination-classification
description: Use when you have promoter and enhancer DNA sequences (e.g. the EPdata
  folder) and need to test whether a promoter-identification model can classify enhancers
  as negatives on a count-balanced, cell-line-specific benchmark.
when_to_use_negative:
- Your goal is de novo promoter prediction across species rather than the promoter-vs-enhancer
  discrimination experiment — use the 12 'data' folder datasets and standard iPro-WAEL
  workflow instead.
- You need to retrain or tune model parameters — this skill explicitly keeps the promoter-trained
  parameters unchanged, so it is wrong for hyperparameter optimization studies.
- Your sequences are not human cell-line promoter/enhancer pairs; the EPdata experiment
  is defined for specific human cell lines selected via the 'cell_lines' variable.
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: iPro-WAEL
  role: Weighted average ensemble learning framework (RF + CNN) providing the frozen
    models and prediction pipeline for promoter/enhancer discrimination
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: feature_code.py
  role: Extracts word2vec-based sequence features that feed both base models
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: RF.py
  role: Random forest base model (scikit-learn) applied to encoded features without
    parameter retuning
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: CNN.py
  role: CNN base model (keras 2.3.1 with tensorflow 2.0.0) applied to encoded features
    without parameter retuning
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: Weighted_average.py
  role: Weighted average algorithm combining RF and CNN outputs into final predictions,
    invoked via main.py
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: iLearnPlus
  role: Extracts the non-word2vec sequence features used alongside word2vec encodings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:8d4340256965a717a3f726c9174b0c53e2a894fc22ad48c9172cf83d7abc82ef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  repo_url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  tools:
  - CNN.py
  - Python
  - RF.py
  - Weighted_average.py
  - feature_code.py
  - h5py
  - iLearnPlus
  - iPro-WAEL
  - keras
  - numpy
  - sklearn
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# promoter-enhancer-discrimination-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reuses the pre-trained iPro-WAEL weighted-average ensemble (random forest + CNN) to discriminate promoters from enhancers on a balanced dataset, testing whether promoter-trained parameters generalize to the enhancer-vs-promoter task without any retuning.

## When to use

Use when you have promoter and enhancer DNA sequences (e.g. the EPdata folder) and need to test whether a promoter-identification model can classify enhancers as negatives on a count-balanced, cell-line-specific benchmark.

## When NOT to use

- Your goal is de novo promoter prediction across species rather than the promoter-vs-enhancer discrimination experiment — use the 12 'data' folder datasets and standard iPro-WAEL workflow instead.
- You need to retrain or tune model parameters — this skill explicitly keeps the promoter-trained parameters unchanged, so it is wrong for hyperparameter optimization studies.
- Your sequences are not human cell-line promoter/enhancer pairs; the EPdata experiment is defined for specific human cell lines selected via the 'cell_lines' variable.

## Inputs

- Enhancer and promoter DNA sequences from the 'EPdata' folder (independent test sets and training sets, first half labeled 1, second half labeled 0)
- Cell line identifier to set as the 'cell_lines' variable in main.py
- Trained iPro-WAEL models from the 'model' folder (12 datasets, seven species)

## Outputs

- Balanced promoter-vs-enhancer predictions from the RF and CNN base models
- Final weighted-average ensemble promoter/enhancer predictions
- performance_metrics.csv containing discrimination metrics on the independent test set (comparable to Supplementary Table S19)

## How to apply

Load enhancer and promoter sequences from the 'EPdata' folder, labeling the first half of each file as positive (1) and the second half as negative (0), and subsample enhancers so positive and negative counts are equal per cell line. Encode sequences into word2vec-based features with feature_code.py and extract the remaining sequence features with iLearnPlus. Manually set the 'cell_lines' variable in main.py to the target cell line, then run RF.py (scikit-learn random forest) and CNN.py (keras 2.3.1 / tensorflow 2.0.0) on the encoded features without retuning any parameters, since the frozen parameters are exactly what tests cross-task generalization. Combine RF and CNN outputs with Weighted_average.py invoked through main.py (`python main.py`) to obtain final promoter/enhancer predictions. Compute discrimination metrics on the independent test set, write them to performance_metrics.csv, and verify agreement with the promoter-vs-enhancer performance in Supplementary Table S19.

## Related tools

- **iPro-WAEL** (Weighted average ensemble learning framework (RF + CNN) providing the frozen models and prediction pipeline for promoter/enhancer discrimination) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **feature_code.py** (Extracts word2vec-based sequence features that feed both base models) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **RF.py** (Random forest base model (scikit-learn) applied to encoded features without parameter retuning) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **CNN.py** (CNN base model (keras 2.3.1 with tensorflow 2.0.0) applied to encoded features without parameter retuning) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **Weighted_average.py** (Weighted average algorithm combining RF and CNN outputs into final predictions, invoked via main.py) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **iLearnPlus** (Extracts the non-word2vec sequence features used alongside word2vec encodings)

## Examples

```
python feature_code.py  # extract word2vec features; then set cell_lines in main.py and run: python main.py
```

## Evaluation signals

- Positive and negative counts are exactly equal per cell line after subsampling enhancers to match promoter counts.
- No model parameters were retuned: RF.py and CNN.py run with the promoter-trained parameters from the 'model' folder unchanged.
- Discrimination metrics computed on the independent test set agree with the promoter-vs-enhancer performance reported in Supplementary Table S19.
- Metrics are written to performance_metrics.csv with the expected per-cell-line breakdown from the 'cell_lines' variable.
- iPro-WAEL achieves exceptionally high performance distinguishing promoters from enhancers despite remarkably similar sequence architectures.

## Limitations

- The feature file for the human dataset is too large to upload to GitHub, so feature extraction must be reproduced locally before running the ensemble.
- The 'cell_lines' variable must be manually modified in main.py to switch cell lines; no automated per-cell-line loop is provided.
- Performance relies on balanced negative subsampling; results assume enhancers are subsampled to match promoter counts per cell line.

## Evidence
