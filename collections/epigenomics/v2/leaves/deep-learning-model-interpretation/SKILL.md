---
name: deep-learning-model-interpretation
description: Use when a trained deep learning model over biological sequences (e.g.,
  one-hot encoded sgRNA matrices) needs feature-level interpretation, such as identifying
  which nucleotides at which positions contribute most to predicted sgRNA activity.
when_to_use_negative:
- The model is a tree-based ensemble over tabular epigenetic features (nucleosome
  occupancy, chromatin accessibility, DNA methylation, gene expression) — use Tree
  SHAP/TreeExplainer instead of DeepExplainer
- The goal is raw sgRNA activity prediction rather than interpreting which sequence
  features drive predictions
- Only a handful of sgRNAs are available — the ~1000-sgRNA averaging is needed to
  obtain stable mean nucleotide-contribution profiles
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: DeepSHAP
  role: DeepExplainer function computes per-base, per-position SHAP attribution values
    over one-hot encoded sgRNA sequences from the trained EpiCas-DL CNN models
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: The trained deep learning framework (sequence and epi CNN models for CRISPRoff/CRISPRa)
    whose learned sgRNA-sequence features are being interpreted
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for the EpiCas-DL deep learning models that DeepSHAP explains
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TreeExplainer
  role: Interprets the XGBoost component by computing mean |SHAP| feature-importance
    values on the training dataset
- name: XGBoost
  role: Algorithm whose SHAP values link epigenetic feature importances for the epi
    models
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:3204becf83611f9dd63ee7b02f3736c084199cd490b8d7f940501b3b1dc5a4f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - DeepSHAP
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  - TreeExplainer
  - XGBoost
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# deep-learning-model-interpretation

## Summary

Interprets trained EpiCas-DL deep learning models using DeepSHAP sequence attributions to identify position-specific nucleotide preferences (e.g., guanine favoring at PAM-adjacent positions) that drive predicted sgRNA efficiency for CRISPR-mediated epigenome editing.

## When to use

Use when a trained deep learning model over biological sequences (e.g., one-hot encoded sgRNA matrices) needs feature-level interpretation, such as identifying which nucleotides at which positions contribute most to predicted sgRNA activity.

## When NOT to use

- The model is a tree-based ensemble over tabular epigenetic features (nucleosome occupancy, chromatin accessibility, DNA methylation, gene expression) — use Tree SHAP/TreeExplainer instead of DeepExplainer
- The goal is raw sgRNA activity prediction rather than interpreting which sequence features drive predictions
- Only a handful of sgRNAs are available — the ~1000-sgRNA averaging is needed to obtain stable mean nucleotide-contribution profiles

## Inputs

- Trained EpiCas-DL CNN models (CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi; TensorFlow 2.4.1 backend)
- One-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels: 9 bp upstream, 23 bp target, 8 bp downstream)
- Randomly selected subset of ~1000 sgRNAs from the CRISPRoff or CRISPRa screening datasets

## Outputs

- Per-base, per-position SHAP attribution values for each interpreted sgRNA
- Mean nucleotide-contribution profiles across all 40 sequence positions
- Histogram of mean SHAP contribution per nucleotide channel at PAM-proximal positions (e.g., positions 20/21)

## How to apply

Load the trained CRISPRoff or CRISPRa EpiCas-DL CNN models (TensorFlow 2.4.1 backend) together with the one-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels covering 9 bp upstream, 23 bp target, and 8 bp downstream). Apply DeepSHAP's DeepExplainer function to compute per-base, per-position SHAP attribution values for a random sample of 1000 sgRNAs per model. Average the SHAP values across the 1000 sgRNAs to obtain mean nucleotide-contribution profiles at each of the 40 sequence positions, then extract the profiles at PAM-proximal positions (20 and 21) and visualize them as a histogram of mean SHAP contribution per nucleotide channel. The rationale is that SHAP attributions expose which sequence features the trained model relies on, revealing biological rules such as the PAM-proximal guanine preference that sequence-based efficiency models implicitly learn. For tree-based components of the pipeline (e.g., XGBoost over epigenetic features), use TreeExplainer with mean |SHAP| values on the training dataset instead.

## Related tools

- **DeepSHAP** (DeepExplainer function computes per-base, per-position SHAP attribution values over one-hot encoded sgRNA sequences from the trained EpiCas-DL CNN models) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (The trained deep learning framework (sequence and epi CNN models for CRISPRoff/CRISPRa) whose learned sgRNA-sequence features are being interpreted) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for the EpiCas-DL deep learning models that DeepSHAP explains) — https://github.com/yangqianq/EpiCas-DL
- **TreeExplainer** (Interprets the XGBoost component by computing mean |SHAP| feature-importance values on the training dataset)
- **XGBoost** (Algorithm whose SHAP values link epigenetic feature importances for the epi models)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- The guanine channel exhibits the highest mean positive SHAP contribution at PAM-proximal positions 20/21 for highly efficient sgRNAs, reproducing the reported PAM-proximal guanine preference in the CRISPRoff model
- Guanine is also favored at position 20 in the CRISPRa model's SHAP sequence attributions
- Mean SHAP profiles are computed across 1000 randomly selected sgRNAs per model, so re-sampling should yield qualitatively consistent nucleotide-contribution histograms
- Attribution matrices have the same shape as the input one-hot encoding (40 positions × 4 A/T/G/C channels) with per-position contributions summing to the model output difference per DeepSHAP's additivity property

## Limitations

- Attributions are model-specific: features the CNN learned may not transfer across CRISPRoff/CRISPRi/CRISPRa systems, as system differences might affect prediction performance
- Interpretation depends on trained-model quality; available CRISPRoff screening data were mainly from HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization of inferred sequence rules
- Incomplete epigenetic feature information limited model performance, so SHAP-derived feature importance may reflect imputation or feature gaps

## Evidence

- [methods] we further identified features contributing to the performance of EpiCas-DL by DeepSHAP: "we further identified features contributing to the performance of EpiCas-DL by DeepSHAP [27]"
- [methods] One-hot encoding of sgRNA sequences: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] TensorFlow backend: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [methods] TreeExplainer mean |SHAP|: "Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer"
- [readme] README model availability: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
