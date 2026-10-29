---
name: crisproff-crispra-model-attribution-averaging
description: Use when you have a trained sequence-based CNN for sgRNA activity (e.g.,
  EpiCas-DL CRISPRoff/CRISPRa models) and need per-base, per-position feature attributions
  to explain sequence determinants of efficiency.
when_to_use_negative:
- You want feature attributions for tabular epigenetic features (nucleosome occupancy,
  methylation, accessibility) — the paper uses Tree SHAP with XGBoost for those, not
  DeepExplainer on sequence matrices
- You only need sgRNA efficiency predictions (use example.py prediction mode) rather
  than model interpretation
- Your model input is not one-hot encoded sequence with the 9/23/8 bp layout — attribution
  positions will not align to PAM-proximal coordinates
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: DeepSHAP
  role: DeepExplainer computes per-base, per-position SHAP attribution values on the
    CNN sequence models
- name: TensorFlow 2.4.1
  role: Backend for the trained EpiCas-DL CNN models being interpreted
- name: EpiCas-DL
  role: Source framework providing the CRISPRoff and CRISPRa trained models and one-hot
    encoded sgRNA inputs
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:8fcb61c23c80329c875ea98485569c777ec2e9f761ecf76ee894ebeeecb18ada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - DeepSHAP
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# crisproff-crispra-model-attribution-averaging

## Summary

Applies DeepSHAP's DeepExplainer to the trained EpiCas-DL CNN models (CRISPRoff and CRISPRa, TensorFlow 2.4.1 backend) over one-hot encoded 40-bp sgRNA matrices, averaging per-base SHAP attributions across 1000 sgRNAs to expose position-specific nucleotide preferences such as the PAM-proximal guanine bias.

## When to use

Use when you have a trained sequence-based CNN for sgRNA activity (e.g., EpiCas-DL CRISPRoff/CRISPRa models) and need per-base, per-position feature attributions to explain sequence determinants of efficiency.

## When NOT to use

- You want feature attributions for tabular epigenetic features (nucleosome occupancy, methylation, accessibility) — the paper uses Tree SHAP with XGBoost for those, not DeepExplainer on sequence matrices
- You only need sgRNA efficiency predictions (use example.py prediction mode) rather than model interpretation
- Your model input is not one-hot encoded sequence with the 9/23/8 bp layout — attribution positions will not align to PAM-proximal coordinates

## Inputs

- Trained EpiCas-DL CRISPRoff_seq and CRISPRa_seq CNN models (TensorFlow 2.4.1)
- One-hot encoded 40-bp sgRNA sequence matrices (four A/T/G/C channels: 9 bp upstream, 23 bp target, 8 bp downstream)
- Randomly selected set of 1000 sgRNAs per model (from CRISPRoff and CRISPRa screening datasets)

## Outputs

- Per-base, per-position SHAP attribution values for each of the 1000 sgRNAs
- Mean SHAP contribution profiles per nucleotide channel at each of the 40 sequence positions
- Histogram of mean SHAP contribution per nucleotide at PAM-proximal positions (20 and 21)

## How to apply

Load the trained CRISPRoff and CRISPRa EpiCas-DL CNN models (TensorFlow 2.4.1 backend) together with the one-hot encoded 40-bp sgRNA matrices (four A/T/G/C channels spanning 9 bp upstream, 23 bp target, 8 bp downstream context). Apply DeepSHAP's DeepExplainer to compute per-base, per-position SHAP attribution values for a random sample of 1000 sgRNAs per model. Average the SHAP values across the 1000 sgRNAs to obtain mean nucleotide-contribution profiles at each of the 40 sequence positions, then extract the profiles at PAM-proximal positions (20 and 21) and display them as a histogram of mean SHAP contribution per nucleotide channel. The rationale is that individual sgRNA attributions are noisy, so cross-sgRNA averaging isolates systematic position-specific base preferences the CNN learned.

## Related tools

- **DeepSHAP** (DeepExplainer computes per-base, per-position SHAP attribution values on the CNN sequence models)
- **TensorFlow 2.4.1** (Backend for the trained EpiCas-DL CNN models being interpreted)
- **EpiCas-DL** (Source framework providing the CRISPRoff and CRISPRa trained models and one-hot encoded sgRNA inputs) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- The guanine channel shows the highest mean positive SHAP contribution at PAM-proximal position 20 in both CRISPRoff and CRISPRa models, and at position 24 in the CRISPRoff model, reproducing the paper's reported guanine preference
- Mean profiles are computed as averages over exactly 1000 randomly selected sgRNAs per model; running with a different random seed should preserve the qualitative PAM-proximal G ranking
- Attribution matrix shape matches the input encoding: 4 nucleotide channels x 40 sequence positions per sgRNA
- Interpretation is restricted to the sequence models; Tree SHAP on the training dataset is the expected method for the tabular-feature models

## Limitations

- CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, increasing overfitting risk and limiting generalization of learned sequence preferences
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance, and sequence attributions reflect only what the CNN captured
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, which propagates into averaged attributions

## Evidence

- [methods] we further identified features contributing to the performance of EpiCas-DL by DeepSHAP: "we further identified features contributing to the performance of EpiCas-DL by DeepSHAP [27]"
- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [readme] There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you.: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
- [abstract] EpiCas-DL also identifies both epigenetic and sequence features that affect sgRNA efficacy in gene silencing and activation: "EpiCas-DL also identifies both epigenetic and sequence features that affect sgRNA efficacy in gene silencing and activation"
