---
name: spearman-correlation-regression-scoring
description: Use when a model outputs continuous sgRNA activity/efficiency predictions
  and you must quantify rank agreement with measured phenotype or activity scores,
  e.g. validating EpiCas-DL on independent genome-wide screening datasets.
when_to_use_negative:
- The prediction task is purely binary classification with no continuous predicted
  scores — ROC-AUC alone suffices and rank correlation is not the target metric
- Input sgRNAs are from the training distribution rather than independent cell-line/dataset
  validation, so the generalization claim being tested does not apply
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Backend machine-learning library used to compute Spearman correlation and
    ROC-AUC evaluation metrics
- name: TensorFlow 2.4.1
  role: Runs inference with the pretrained CRISPRoff_seq/CRISPRoff_epi and CRISPRa_seq/CRISPRa_epi
    CNN models that produce the scores being scored
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Deep learning framework whose regression predictions are evaluated by Spearman
    correlation
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Scripting environment for the evaluation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:d0dbc10778727b64246d2794fbb3fe8d540edebcbeddf03a2baa1c3d184e55b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spearman-correlation-regression-scoring

## Summary

Evaluates regression predictions of sgRNA editing efficiency by computing Spearman rank correlation (alongside classification ROC-AUC) with scikit-learn, as used to score EpiCas-DL CNN models on independent CRISPRi/CRISPRoff/CRISPRa screening datasets.

## When to use

Use when a model outputs continuous sgRNA activity/efficiency predictions and you must quantify rank agreement with measured phenotype or activity scores, e.g. validating EpiCas-DL on independent genome-wide screening datasets.

## When NOT to use

- The prediction task is purely binary classification with no continuous predicted scores — ROC-AUC alone suffices and rank correlation is not the target metric
- Input sgRNAs are from the training distribution rather than independent cell-line/dataset validation, so the generalization claim being tested does not apply

## Inputs

- Predicted sgRNA activity/efficiency scores from the trained EpiCas-DL regression model (CRISPRoff_epi/CRISPRa_epi or seq-only variants)
- Observed sgRNA activity scores or phenotype scores from screening datasets (e.g. CRISPRi_activityscore, hCRISPRi-v2, CRISPRi_genome, CRISPRi_K562, CRISPRoff_genome, CRISPRa_activityscore)
- Binary high-activity labels derived from threshold criteria (phenotype score < -0.1; activity score > 0.75 for activityscore datasets)

## Outputs

- Spearman correlation coefficient between predicted and measured sgRNA efficiency per model/dataset
- Companion ROC-AUC values for the classification schema
- Metrics table written to independent_validation_metrics.csv

## How to apply

Run inference with the trained EpiCas-DL model (TensorFlow 2.4.1) on held-out or independent sgRNAs encoded as 11 x 40 one-hot sequence plus min-max normalized epigenetic channel matrices. For the regression schema, compute the Spearman correlation coefficient between predicted and observed activity scores using scikit-learn (v0.24.2); for the paired classification schema, label sgRNAs as high-activity ('1') using the reported criteria (phenotype scores below -0.1 as highly active; activity scores above 0.75 for the activityscore datasets) and compute ROC-AUC. Apply the CRISPRoff/CRISPRi models to the five independent gene-silencing datasets and the CRISPRa model to CRISPRa_activityscore, then write all metrics to a CSV (e.g. independent_validation_metrics.csv). A correct application reproduces the reported ranking where EpiCas-DL leads other models (overall ROC-AUC 0.746 on combined independent data) with a higher Spearman correlation.

## Related tools

- **scikit-learn** (Backend machine-learning library used to compute Spearman correlation and ROC-AUC evaluation metrics)
- **TensorFlow 2.4.1** (Runs inference with the pretrained CRISPRoff_seq/CRISPRoff_epi and CRISPRa_seq/CRISPRa_epi CNN models that produce the scores being scored) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Deep learning framework whose regression predictions are evaluated by Spearman correlation) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Scripting environment for the evaluation workflow)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- independent_validation_metrics.csv exists and contains ROC-AUC values within [0, 1]
- Reproduced Spearman correlation values are consistent in sign and magnitude with reported high performance on independent validation comparisons (Fig. 3b)
- Combined independent-dataset ROC-AUC for EpiCas-DL matches the reported 0.746 (Fig. 3a)
- High-activity labels were assigned with the correct thresholds: phenotype score < -0.1, activity score > 0.75 for activityscore datasets
- EpiCas-DL ranks above comparison models (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) on both Spearman correlation and ROC-AUC

## Limitations

- Differences between CRISPRoff and CRISPRi systems might affect prediction performance of EpiCas-DL in the testing datasets
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, affecting correlation values
- Incomplete epigenetic feature information (only nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) limited model performance

## Evidence

- [methods] Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [results] ROC curve AUC or Spearman correlation computed on 90/10 split: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [readme] Four EpiCas-DL models provided: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
- [results] Independent validation ROC-AUC of 0.746: [evidence span withheld — anchor status: not_found]
