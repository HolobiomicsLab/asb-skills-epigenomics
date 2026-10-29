---
name: roc-auc-classification-scoring
description: Use when evaluating binary sgRNA high-activity classifiers (e.g., labeled
  via phenotype scores below -0.1 or activity scores above 0.75) and you need a threshold-independent
  ROC-AUC metric to compare prediction models.
when_to_use_negative:
- The evaluation goal is ordinal/rank agreement on continuous activity scores rather
  than binary classification — use Spearman correlation instead, as the paper does
  for its regression schema (Fig. 3b).
- Ground-truth labels are not available for the held-out sgRNAs, so a supervised classification
  metric such as ROC-AUC cannot be computed.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Backend used to compute the ROC-AUC (and Spearman correlation) evaluation
    metrics on model predictions
- name: TensorFlow 2.4.1
  role: Backend for the EpiCas-DL CNN models (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi)
    whose classification output is scored with ROC-AUC
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Deep-learning framework whose sgRNA-activity predictions are evaluated with
    ROC-AUC during independent-dataset validation
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Runtime environment for the evaluation scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:ab6c1fdbbfcee630b971394c03369fc8f1e67512a37c7cada9098a6ebfbd446e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# roc-auc-classification-scoring

## Summary

Scores sgRNA-activity classification performance with the area under the ROC curve (ROC-AUC) using scikit-learn, as done in EpiCas-DL to benchmark deep-learning models against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr on independent CRISPRi/CRISPRoff/CRISPRa screening datasets.

## When to use

Use when evaluating binary sgRNA high-activity classifiers (e.g., labeled via phenotype scores below -0.1 or activity scores above 0.75) and you need a threshold-independent ROC-AUC metric to compare prediction models.

## When NOT to use

- The evaluation goal is ordinal/rank agreement on continuous activity scores rather than binary classification — use Spearman correlation instead, as the paper does for its regression schema (Fig. 3b).
- Ground-truth labels are not available for the held-out sgRNAs, so a supervised classification metric such as ROC-AUC cannot be computed.

## Inputs

- Predicted sgRNA activity scores/probabilities from a CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi model
- Binary ground-truth labels for sgRNAs (phenotype score < -0.1 for silencing; activity score > 0.75 for activityscore datasets)
- sgRNA 11 x 40 input matrices (one-hot sequence + min-max normalized nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression channels)

## Outputs

- ROC-AUC value per dataset (e.g., 0.746 combined ROC-AUC, Fig. 3a)
- Metrics table such as independent_validation_metrics.csv

## How to apply

Label sgRNAs as highly active ('1') using the reported criteria: phenotype scores below -0.1 for gene-silencing datasets and activity scores above 0.75 for the activityscore datasets (e.g., CRISPRa_activityscore with 2,779 sgRNAs on 236 genes). Run inference with the trained CRISPRoff_seq/CRISPRoff_epi or CRISPRa_seq/CRISPRa_epi EpiCas-DL CNN models (TensorFlow 2.4.1) on sgRNAs encoded as 11 x 40 one-hot plus min-max normalized epigenetic input matrices, then compute ROC-AUC on the predicted probabilities with scikit-learn 0.24.2. Apply the same schema to the combined five independent gene-silencing datasets (CRISPRi_activityscore, CRISPRi_K562, CRISPRoff_genome, hCRISPRi-v2, CRISPRi_genome) and write metrics to a CSV such as independent_validation_metrics.csv. Interpret results by comparison against competing algorithms, expecting the reported combined-independent-dataset ROC-AUC of 0.746 for EpiCas-DL (Fig. 3a).

## Related tools

- **scikit-learn** (Backend used to compute the ROC-AUC (and Spearman correlation) evaluation metrics on model predictions)
- **TensorFlow 2.4.1** (Backend for the EpiCas-DL CNN models (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi) whose classification output is scored with ROC-AUC) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Deep-learning framework whose sgRNA-activity predictions are evaluated with ROC-AUC during independent-dataset validation) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Runtime environment for the evaluation scripts)

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_true, model.predict(X_11x40))
```

## Evaluation signals

- Metrics file (e.g., independent_validation_metrics.csv) exists and contains the reproduced ROC-AUC values
- All ROC-AUC values fall within [0, 1]
- Combined independent-dataset ROC-AUC is in the reported high-performance range, ~0.746 as in Fig. 3a
- EpiCas-DL ROC-AUC equals or exceeds the competing algorithms (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) on the same labeled datasets
- Labeling thresholds applied as specified: phenotype score < -0.1 for silencing datasets, activity score > 0.75 for activityscore datasets

## Limitations

- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in the testing datasets
- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, affecting label quality

## Evidence

- [results] On the combined independent datasets, EpiCas-DL outperformed the other models with an overall ROC-AUC of 0.746 (Fig. 3a): [evidence span withheld — anchor status: not_found]
- [results] Independent validation split and metric choice: "we combined 90 % of the five independent datasets for model training and examined the model performance on the remaining 10 % of the data"
- [methods] Tooling for computing metrics: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [readme] Available pretrained models to score: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
