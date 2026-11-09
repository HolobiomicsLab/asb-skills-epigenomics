---
name: shap-sequence-attribution-analysis
description: Use when you have a trained EpiCas-DL model (TensorFlow 2.4.1 backend)
  plus one-hot encoded 40-bp sgRNA sequences (9 bp upstream, 23 bp target, 8 bp downstream)
  and want position-specific nucleotide contributions explaining sgRNA efficiency
  predictions.
when_to_use_negative:
- Input is raw sgRNA sequences or activity scores rather than a trained EpiCas-DL
  model with one-hot encoded inputs — SHAP attribution requires an existing fitted
  model
- Interpreting the machine-learning (Gradient Boosting/Random Forest) models — use
  Tree SHAP via TreeExplainer instead of DeepExplainer
- Screening datasets other than the CRISPRoff/CRISPRa epigenome-editing contexts,
  where the reported nucleotide-preference findings may not transfer
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: DeepSHAP
  role: DeepExplainer applied to the EpiCas-DL CNN to compute per-base, per-position
    SHAP attribution values
- name: TensorFlow 2.4.1
  role: Backend for the trained EpiCas-DL deep learning models that DeepSHAP attributes
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: The sgRNA-activity prediction framework whose trained CRISPRoff/CRISPRa models
    are being interpreted
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Language for the custom interpretation scripts in the EpiCas-DL repository
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:1fdd397e3d70adc272967250fa9feb5a57fd948139d26782dcc89588843cc9ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - DeepSHAP
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# shap-sequence-attribution-analysis

## Summary

Interprets the trained EpiCas-DL CNN models by computing DeepSHAP per-base, per-position attribution values over one-hot encoded 40-bp sgRNA sequence matrices, to reveal position-specific nucleotide preferences (e.g. PAM-proximal guanine favoring) that drive predicted sgRNA efficiency in CRISPRoff/CRISPRa epigenome editing.

## When to use

Use when you have a trained EpiCas-DL model (TensorFlow 2.4.1 backend) plus one-hot encoded 40-bp sgRNA sequences (9 bp upstream, 23 bp target, 8 bp downstream) and want position-specific nucleotide contributions explaining sgRNA efficiency predictions.

## When NOT to use

- Input is raw sgRNA sequences or activity scores rather than a trained EpiCas-DL model with one-hot encoded inputs — SHAP attribution requires an existing fitted model
- Interpreting the machine-learning (Gradient Boosting/Random Forest) models — use Tree SHAP via TreeExplainer instead of DeepExplainer
- Screening datasets other than the CRISPRoff/CRISPRa epigenome-editing contexts, where the reported nucleotide-preference findings may not transfer

## Inputs

- Trained EpiCas-DL CNN models (CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi) built on TensorFlow 2.4.1
- One-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels: 9 bp upstream, 23 bp target, 8 bp downstream)
- Randomly selected subset of 1000 sgRNAs from the CRISPRoff and CRISPRa screening datasets

## Outputs

- Per-base, per-position SHAP attribution values for each sgRNA
- Mean nucleotide-contribution profiles at each of the 40 sequence positions
- Histograms of mean SHAP contribution per nucleotide at PAM-proximal positions (20/21) showing the guanine preference

## How to apply

Load the trained CRISPRoff and/or CRISPRa EpiCas-DL models with their one-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels: 9 bp upstream, 23 bp target, 8 bp downstream). Apply DeepSHAP's DeepExplainer to the CNN to compute per-base, per-position SHAP attributions for a random selection of 1000 sgRNAs per model. Average the SHAP values across the 1000 sgRNAs to obtain mean nucleotide-contribution profiles at each of the 40 positions. Extract profiles at PAM-adjacent positions 20/21 and present them as a histogram of mean SHAP contribution per nucleotide channel; interpret feature importance analogously to the Tree SHAP analysis used for the epigenetic features.

## Related tools

- **DeepSHAP** (DeepExplainer applied to the EpiCas-DL CNN to compute per-base, per-position SHAP attribution values)
- **TensorFlow 2.4.1** (Backend for the trained EpiCas-DL deep learning models that DeepSHAP attributes) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (The sgRNA-activity prediction framework whose trained CRISPRoff/CRISPRa models are being interpreted) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Language for the custom interpretation scripts in the EpiCas-DL repository) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
git clone https://github.com/yangqianq/EpiCas-DL.git; python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- The guanine channel shows the highest mean positive SHAP contribution at PAM-adjacent positions 20/24 for highly efficient sgRNAs in the CRISPRoff model, reproducing the paper's PAM-proximal guanine preference
- The guanine channel is favored at position 20 in the CRISPRa model, confirming consistency across the two models
- Attributions cover all 40 sequence positions (9 bp upstream, 23 bp target, 8 bp downstream) with four nucleotide channels each
- Mean profiles are stable because SHAP values are averaged over 1000 randomly selected sgRNAs rather than a single sequence

## Limitations

- Nucleotide-preference conclusions are specific to the CRISPRoff and CRISPRa screening datasets; the paper notes differences between CRISPRoff and CRISPRi systems might affect prediction performance
- CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization of the learned sequence features
- The paper notes gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, which propagates into learned attributions

## Evidence

- [methods] DeepSHAP applied to EpiCas-DL CNN: "we further identified features contributing to the performance of EpiCas-DL by DeepSHAP [27]"
- [methods] One-hot encoding of sgRNA sequences: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] TensorFlow backend for the deep models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [readme] Repository provides trained models: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
