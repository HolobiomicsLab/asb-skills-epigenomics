---
name: tfbs-motif-importance-ranking
description: Use when you have trained cell-specific enhancer models (e.g. Enhancer-MDLF)
  plus sequence-derived motif features and need to identify which TFBS motifs drive
  predictions in each cell line.
when_to_use_negative:
- The goal is enhancer or enhancer-promoter prediction itself (main.py), not post-hoc
  model interpretation of motif features
- The goal is cross-cell-line prediction via transfer learning rather than per-cell-line
  motif interpretability
- Input is already an aggregated SHAP attribution or importance table rather than
  cell-line FASTA sequences requiring motif feature extraction
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning framework whose motif-module features and trained
    cell-line models are interpreted with SHAP
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: motif_find.py
  role: Extracts motif features by scanning enhancer sequences against HOCOMOCO Human
    v11 PWMs and p-value thresholds, producing the SHAP-interpreted input features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: SHAP
  role: Model interpretation framework used to compute per-feature attributions over
    the motif-module inputs of the trained Enhancer-MDLF models
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: tensorflow
  role: Framework (v2.2.0) in which the Enhancer-MDLF models loaded for SHAP interpretation
    are implemented
- name: HOCOMOCO Human v11
  role: Source of the motif PWMs and p-value threshold scores used to construct the
    motif features being ranked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:f42453ae5d17048c395f95f5d81e20bc1dc595cc264efc215c2b8191d1ed4fd4
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
  - HOCOMOCO Human v11
  - Python
  - SHAP
  - TensorFlow
  - motif_find.py
  - numpy
  - scikit-learn
  - tensorflow
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tfbs-motif-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks transcription factor binding site (TFBS) motifs by importance within enhancer regions by computing SHAP attributions over the motif-module input features of the Enhancer-MDLF deep learning model, per human cell line.

## When to use

Use when you have trained cell-specific enhancer models (e.g. Enhancer-MDLF) plus sequence-derived motif features and need to identify which TFBS motifs drive predictions in each cell line.

## When NOT to use

- The goal is enhancer or enhancer-promoter prediction itself (main.py), not post-hoc model interpretation of motif features
- The goal is cross-cell-line prediction via transfer learning rather than per-cell-line motif interpretability
- Input is already an aggregated SHAP attribution or importance table rather than cell-line FASTA sequences requiring motif feature extraction

## Inputs

- Independent enhancer test set FASTA files and labels for eight human cell lines ('data' folder, e.g. data/test/NHLF.fasta and NHLF_y_test.txt)
- Pre-trained or trained Enhancer-MDLF models for the eight cell lines ('model' folder)
- Position weight matrices (PWM) and p-value threshold scores from the HOCOMOCO Human v11 database ('motif' folder)
- Motif-module input features extracted with motif_find.py

## Outputs

- SHAP force plot interaction diagrams for the eight cell lines ('force-plot' directory)
- Per-cell-line ranking of TFBS motifs by aggregated SHAP importance (top motifs, e.g. BACH1 in HUVEC, GATA1 in K562, SP1 across cell lines)

## How to apply

Load the independent enhancer test set FASTA files and labels for each of the eight cell lines from the 'data' folder, along with the pre-trained or trained Enhancer-MDLF models from the 'model' folder. Build the motif-module input features with motif_find.py, which scans sequences against the position weight matrices (PWM) and p-value threshold scores from the HOCOMOCO Human v11 database in the 'motif' folder. Run SHAP.py to compute SHAP values over the motif-module features for each cell line's model, and save the resulting SHAP force plot interaction diagrams to the 'force-plot' directory. Aggregate SHAP attributions per motif feature to rank TFBS motifs by importance per cell line; the reported reference rankings include BACH1 emerging exclusively among the top 50 important features in HUVEC, GATA1 in K562, and SP1 ranking highly across cell lines.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning framework whose motif-module features and trained cell-line models are interpreted with SHAP) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **motif_find.py** (Extracts motif features by scanning enhancer sequences against HOCOMOCO Human v11 PWMs and p-value thresholds, producing the SHAP-interpreted input features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **SHAP** (Model interpretation framework used to compute per-feature attributions over the motif-module inputs of the trained Enhancer-MDLF models) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **tensorflow** (Framework (v2.2.0) in which the Enhancer-MDLF models loaded for SHAP interpretation are implemented)
- **HOCOMOCO Human v11** (Source of the motif PWMs and p-value threshold scores used to construct the motif features being ranked)

## Examples

```
python motif_find.py --input_file data/test/HUVEC.fasta --cell_line HUVEC --set test; python SHAP.py
```

## Evaluation signals

- The BACH1 motif appears exclusively among the top 50 important features in the HUVEC cell line, consistent with BACH1's known binding in the HO-1 enhancer
- GATA1 ranks highly in K562, and SP1 ranks highly across all eight cell lines
- A force plot interaction diagram file exists for each of the eight cell lines in the 'force-plot' directory
- The motif feature matrix was produced by motif_find.py scanning against HOCOMOCO Human v11 PWMs with the specified p-value threshold scores
- Per-cell-line rankings are computed from SHAP values aggregated per motif feature on the independent test set, not the training set

## Limitations

- Interpretation depends on the motif-module input features; motifs absent from the HOCOMOCO Human v11 PWM set cannot be ranked
- SHAP rankings reflect the behavior of the trained Enhancer-MDLF model on each cell line's independent test set and are model-dependent rather than direct experimental binding measurements
- The repository environment is specific: Python 3.8, pip 23.3.1, tensorflow 2.2.0, scikit-learn, numpy; other versions may conflict (see requirements.txt)

## Evidence

- [other] Using SHAP to interpret the motif-module features of Enhancer-MDLF, the BACH1 motif emerges exclusively among the top 50 important features in the HUVEC cell line: [evidence span withheld — anchor status: not_found]
