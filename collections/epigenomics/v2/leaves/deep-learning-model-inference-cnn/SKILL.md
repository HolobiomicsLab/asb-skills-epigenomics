---
name: deep-learning-model-inference-cnn
description: Use when sgRNAs need activity predictions from a pretrained CRISPRoff/CRISPRa
  CNN, or when an independent screening dataset must be scored to reproduce validation
  metrics (ROC-AUC, Spearman correlation).
when_to_use_negative:
- Training or hyperparameter optimization of the CNN is the goal (use the 9:1 split
  with 5-fold cross-validation and GPyOpt Bayesian optimization instead of pure inference)
- Interpreting which features drive predictions (use Tree SHAP/DeepSHAP feature identification
  rather than inference-only scoring)
- sgRNAs lack epigenetic feature coverage — the epi models require nucleosome, accessibility,
  methylation, and expression channels; use the seq-only models or note the limitation
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: Pretrained CNN framework (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi
    models) whose saved models are run in inference mode
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for loading and executing the trained CNN models during inference
- name: scikit-learn 0.24.2
  role: Computing ROC-AUC and Spearman correlation on predicted vs. labeled sgRNA
    activities
- name: Python 3.6
  role: Language for the inference and evaluation scripts (with pybedtools 0.9.0 for
    coordinate handling)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:51e09aa09d176fb4ba8e6b5497aa99b61d685f28033de0d26bfe6f026b8f9267
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# deep-learning-model-inference-cnn

## Summary

Run trained EpiCas-DL CNN models (CRISPRoff_seq/CRISPRoff_epi and CRISPRa_seq/CRISPRa_epi) in inference mode over one-hot-encoded sgRNA sequences with min-max normalized epigenetic channels to predict sgRNA silencing or activation activity and compute ROC-AUC and Spearman correlation on independent datasets.

## When to use

Use when sgRNAs need activity predictions from a pretrained CRISPRoff/CRISPRa CNN, or when an independent screening dataset must be scored to reproduce validation metrics (ROC-AUC, Spearman correlation).

## When NOT to use

- Training or hyperparameter optimization of the CNN is the goal (use the 9:1 split with 5-fold cross-validation and GPyOpt Bayesian optimization instead of pure inference)
- Interpreting which features drive predictions (use Tree SHAP/DeepSHAP feature identification rather than inference-only scoring)
- sgRNAs lack epigenetic feature coverage — the epi models require nucleosome, accessibility, methylation, and expression channels; use the seq-only models or note the limitation

## Inputs

- sgRNA sequences (40 nt targets) from genome-wide screening datasets (CRISPRi_activityscore, CRISPRi_K562, CRISPRoff_genome, hCRISPRi-v2, CRISPRi_genome, CRISPRa_activityscore)
- epigenetic feature tracks: nucleosome positioning, chromatin accessibility (ATACdb), DNA methylation, RNA expression per target locus
- high-activity labels (phenotype score < -0.1; activity score > 0.75 for activityscore datasets)
- pretrained EpiCas-DL model files (CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi)

## Outputs

- predicted sgRNA activity scores per target
- ROC-AUC values for the classification schema (e.g., overall ROC-AUC 0.746 on combined independent datasets)
- Spearman correlation coefficients for the regression schema
- independent_validation_metrics.csv

## How to apply

Load the pretrained CRISPRoff_seq/CRISPRoff_epi or CRISPRa_seq/CRISPRa_epi models from the EpiCas-DL repository with TensorFlow 2.4.1. Encode each sgRNA as an 11 x 40 input matrix with one-hot sequence channels and min-max normalized epigenetic channels (nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression); for the epi models also supply chromosome, start/end coordinates, PAM position, transcript, gene, and cell line as in the README example. Label sgRNAs as high-activity using the reported criteria (phenotype scores below -0.1; activity scores above 0.75 for activityscore datasets) and run inference on the combined independent gene-silencing data (CRISPRi_activityscore, CRISPRi_K562, CRISPRoff_genome, hCRISPRi-v2, CRISPRi_genome) and CRISPRa_activityscore. Compute ROC-AUC for classification and Spearman correlation for regression with scikit-learn 0.24.2, writing all metrics to independent_validation_metrics.csv.

## Related tools

- **EpiCas-DL** (Pretrained CNN framework (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi models) whose saved models are run in inference mode) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for loading and executing the trained CNN models during inference)
- **scikit-learn 0.24.2** (Computing ROC-AUC and Spearman correlation on predicted vs. labeled sgRNA activities)
- **Python 3.6** (Language for the inference and evaluation scripts (with pybedtools 0.9.0 for coordinate handling))

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- independent_validation_metrics.csv exists and contains ROC-AUC and Spearman correlation entries
- Reproduced ROC-AUC falls within [0, 1] and matches the reported 0.746 overall on combined independent datasets (Fig. 3a)
- EpiCas-DL Spearman correlation exceeds those of Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr (Fig. 3b)
- Each sgRNA input is an 11 x 40 matrix: one-hot sequence channels plus min-max normalized epigenetic channels
- Inference runs the correct model type: CRISPRoff/CRISPRi model on gene-silencing data, CRISPRa model on CRISPRa_activityscore

## Limitations

- Differences between CRISPRoff and CRISPRi systems might affect prediction performance on testing datasets
- Performance was limited by incomplete epigenetic feature information (only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression considered)
- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization
- Screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy

## Evidence

- [other] Load the pretrained CRISPRoff_seq/CRISPRoff_epi and CRISPRa_seq/CRISPRa_epi EpiCas-DL CNN models from the EpiCas-DL repository using TensorFlow 2.4.1.: [evidence span withheld — anchor status: not_found]
- [methods] One-hot and min-max input encoding: "All the epigenetic features were min–max normalized before model input"
- [results] Independent validation result: "EpiCas-DL outperformed the other models with an overall ROC-AUC of 0.746 (Fig. 3a)"
- [readme] README inference invocation with epigenetic features: "python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057"
- [readme] Available pretrained models: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
