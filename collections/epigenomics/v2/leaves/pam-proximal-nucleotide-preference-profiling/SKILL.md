---
name: pam-proximal-nucleotide-preference-profiling
description: Use when a trained CRISPR sgRNA-efficiency CNN (EpiCas-DL CRISPRoff/CRISPRa
  models) needs per-base, per-position attribution to reveal which nucleotide a model
  favors at PAM-adjacent positions 20/21.
when_to_use_negative:
- Interpreting tabular epigenetic features of the EpiCas-DL model — the paper uses
  Tree SHAP with TreeExplainer on XGBoost for feature identification, not DeepSHAP
  sequence attribution
- Training or performance-evaluating sgRNA efficiency models (e.g., 9:1 train/test
  split, 5-fold cross-validation, Bayesian GPyOpt hyperparameter tuning) rather than
  interpreting a trained model
- sgRNA inputs without a defined PAM-relative coordinate frame, where PAM-adjacent
  positions 20/21 cannot be located in the 40-bp encoding
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DeepSHAP
  role: DeepExplainer function computes per-base, per-position SHAP attribution values
    for the one-hot encoded sgRNA sequences
- name: EpiCas-DL
  role: Trained sgRNA-activity CNN (CRISPRoff/CRISPRa) whose sequence attributions
    are being profiled
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for loading and running the trained EpiCas-DL deep learning models
    during SHAP computation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:240d43f4ca1e3abdfec7adc8d0ef72158f2ad68039b55c712e6ea582ca245d05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - DeepSHAP
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pam-proximal-nucleotide-preference-profiling

## Summary

Computes position-specific nucleotide preferences at PAM-proximal sgRNA positions by applying DeepSHAP (DeepExplainer) to trained EpiCas-DL CNN models over one-hot encoded 40-bp sgRNA matrices, then visualizing mean SHAP contributions per nucleotide channel. It reproduces the paper's finding that guanine is favored adjacent to the PAM for efficient sgRNAs.

## When to use

Use when a trained CRISPR sgRNA-efficiency CNN (EpiCas-DL CRISPRoff/CRISPRa models) needs per-base, per-position attribution to reveal which nucleotide a model favors at PAM-adjacent positions 20/21.

## When NOT to use

- Interpreting tabular epigenetic features of the EpiCas-DL model — the paper uses Tree SHAP with TreeExplainer on XGBoost for feature identification, not DeepSHAP sequence attribution
- Training or performance-evaluating sgRNA efficiency models (e.g., 9:1 train/test split, 5-fold cross-validation, Bayesian GPyOpt hyperparameter tuning) rather than interpreting a trained model
- sgRNA inputs without a defined PAM-relative coordinate frame, where PAM-adjacent positions 20/21 cannot be located in the 40-bp encoding

## Inputs

- Trained EpiCas-DL CNN models (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi) on TensorFlow 2.4.1
- One-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels: 9 bp upstream, 23 bp target, 8 bp downstream)
- Randomly selected 1000 sgRNAs from the CRISPRoff and CRISPRa screening datasets

## Outputs

- Mean nucleotide-contribution profiles (SHAP values averaged over 1000 sgRNAs) at each of 40 sequence positions
- Histogram of mean SHAP contribution per nucleotide channel at PAM-adjacent positions 20 and 21
- Identification of the favored nucleotide at PAM-proximal positions for highly efficient sgRNAs (guanine in CRISPRoff at positions 20/24; guanine at position 20 in CRISPRa)

## How to apply

Load the trained CRISPRoff and CRISPRa EpiCas-DL CNN models on the TensorFlow 2.4.1 backend together with one-hot encoded 40-bp sgRNA matrices (four A/T/G/C channels covering 9 bp upstream, 23 bp target, 8 bp downstream). Apply DeepSHAP's DeepExplainer to compute per-base, per-position SHAP attribution values for randomly selected 1000 sgRNAs from each screening dataset. Average the SHAP values across the 1000 sgRNAs to obtain mean nucleotide-contribution profiles at each of the 40 sequence positions. Extract the profiles at positions 20 and 21 adjacent to the PAM and render them as a histogram of mean SHAP contribution per nucleotide channel. Validate that the guanine channel exhibits the highest mean positive SHAP contribution at the PAM-proximal positions, reproducing the reported guanine preference for highly efficient sgRNAs.

## Related tools

- **DeepSHAP** (DeepExplainer function computes per-base, per-position SHAP attribution values for the one-hot encoded sgRNA sequences)
- **EpiCas-DL** (Trained sgRNA-activity CNN (CRISPRoff/CRISPRa) whose sequence attributions are being profiled) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for loading and running the trained EpiCas-DL deep learning models during SHAP computation)

## Examples

```
from shap import DeepExplainer; explainer = DeepExplainer(model, background); shap_values = explainer.shap_values(sgrna_onehot[:1000])
```

## Evaluation signals

- Guanine channel shows the highest mean positive SHAP contribution at PAM-proximal positions 20/21 in the CRISPRoff model for highly efficient sgRNAs
- Guanine is favored at position 20 in the CRISPRa model, as reported in the paper
- Mean-contribution profile has the expected shape of 40 positions x 4 nucleotide channels matching the one-hot encoding (9 bp upstream + 23 bp target + 8 bp downstream)
- At each sequence position, exactly one nucleotide channel carries the attribution, consistent with the image-like one-hot coding scheme
- Profiles are averaged over the same randomly selected 1000 sgRNAs per model, so re-running the sampling should yield the same PAM-proximal ordering of nucleotide contributions

## Limitations

- Available CRISPRoff screening datasets were relatively small and mainly conducted in HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization of the learned nucleotide preferences
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, which propagates into the SHAP-derived preferences
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance, so the guanine-favoring result is demonstrated for CRISPRoff and CRISPRa models rather than universally

## Evidence

- [methods] we further identified features contributing to the performance of EpiCas-DL by DeepSHAP [27]: "we further identified features contributing to the performance of EpiCas-DL by DeepSHAP [27]"
- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [readme] There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you.: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
