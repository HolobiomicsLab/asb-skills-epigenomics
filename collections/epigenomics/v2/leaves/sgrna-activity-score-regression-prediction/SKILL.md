---
name: sgrna-activity-score-regression-prediction
description: Use when you must predict sgRNA gene-silencing or activation activity
  scores (e.g. CRISPRoff_tiling, 111,682 sgRNAs within ±2.5 kb of TSS of 520 genes)
  from sequence plus cell-type epigenetic tracks, and benchmark regressors by Spearman
  correlation.
when_to_use_negative:
- The prediction target is Cas9 nuclease cutting/indel editing efficiency rather than
  dCas-mediated gene repression or activation (CRISPRoff/CRISPRi/CRISPRa) activity
  scores
- The task is binary on-target efficiency classification (AUC evaluation) rather than
  regression of gene-level normalized activity scores
- The sgRNAs fall outside ±2.5 kb of a TSS or lack the four epigenetic tracks and
  cell-line context, so epigenetic channels cannot be populated (use the CRISPRoff_seq/CRISPRa_seq
  sequence-only models instead)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: EpiCas-DL
  role: Core CNN regression framework that learns sgRNA activity from 11x40 sequence-plus-epigenetic
    input matrices
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for training the EpiCas-DL CNN regressor with 1–5 nt filter groups,
    MaxPooling, and 80/60/40 fully connected layers
- name: scikit-learn 0.24.2
  role: Backend for the comparator algorithms (Gradient Boosting, Random Forest) fitted
    on identical inputs
- name: GPyOpt
  role: Bayesian hyperparameter optimization of the CNN (30 initial random points,
    300 acquisitions)
- name: pybedtools 0.9.0
  role: Custom Python scripting for assembling sgRNA coordinate and epigenetic feature
    inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:5429f0b9c3e56a6916fc59b23efdb6f617a863e5e99cdc443a6c2d7c1d6cb41c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - TensorFlow 2.4.1
  - pybedtools 0.9.0
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sgrna-activity-score-regression-prediction

## Summary

A deep learning regression skill (EpiCas-DL) that predicts gene-repression/activation sgRNA activity from 11x40 encoded sequence-and-epigenetic feature matrices (one-hot sequence plus nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression channels), benchmarked against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr by Spearman correlation, MSE, and cosine similarity.

## When to use

Use when you must predict sgRNA gene-silencing or activation activity scores (e.g. CRISPRoff_tiling, 111,682 sgRNAs within ±2.5 kb of TSS of 520 genes) from sequence plus cell-type epigenetic tracks, and benchmark regressors by Spearman correlation.

## When NOT to use

- The prediction target is Cas9 nuclease cutting/indel editing efficiency rather than dCas-mediated gene repression or activation (CRISPRoff/CRISPRi/CRISPRa) activity scores
- The task is binary on-target efficiency classification (AUC evaluation) rather than regression of gene-level normalized activity scores
- The sgRNAs fall outside ±2.5 kb of a TSS or lack the four epigenetic tracks and cell-line context, so epigenetic channels cannot be populated (use the CRISPRoff_seq/CRISPRa_seq sequence-only models instead)

## Inputs

- sgRNA tiling dataset with gene-level normalized activity scores (e.g. CRISPRoff_tiling: 111,682 sgRNAs within ±2.5 kb of TSS of 520 genes)
- 11x40 encoded feature matrices: 4 one-hot sequence channels plus nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression channels (min-max normalized)
- Genomic coordinates per sgRNA (chromosome, start, end, PAM position, transcript, gene, cell line) for epigenetic feature lookup
- Pretrained model files such as CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi

## Outputs

- Per-sgRNA predicted activity scores (continuous regression output from a linear output layer)
- Per-fold evaluation metrics: Spearman correlation, mean squared error, cosine similarity across 10 cross-validation folds
- Comparison of EpiCas-DL vs. Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr with Student's t-test p-values
- Tree SHAP / DeepSHAP feature attributions for epigenetic and sequence channels

## How to apply

Load a tiling dataset of sgRNAs with gene-level normalized activity scores (phenotype score normalized by the absolute average score of the top 3 sgRNAs per gene) and build 11x40 input matrices: 4 one-hot sequence channels plus epigenetic channels that are min-max normalized. Randomly split data 9:1 into training/testing and run cross-validation (5-fold for hyperparameter search, 10-fold for reported evaluation). Train the EpiCas-DL CNN regressor in TensorFlow 2.4.1 with five filter groups (30 filters each of 1–5 nt), MaxPooling, three fully connected layers (80/60/40 units), dropout 0.4 after batch normalization, ReLU activations, and a linear output layer; tune hyperparameters with GPyOpt Bayesian optimization (30 initial random points, 300 acquisitions). Fit the four comparators (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) via scikit-learn 0.24.2 on identical inputs, then evaluate every model per fold by Spearman correlation, mean squared error, and cosine similarity, and test EpiCas-DL vs. baselines with Student's t-test. For downstream reuse, pretrained CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi models accept a sequence plus chromosome/start/end/PAM/transcript/gene/cell_line coordinates via example.py.

## Related tools

- **EpiCas-DL** (Core CNN regression framework that learns sgRNA activity from 11x40 sequence-plus-epigenetic input matrices) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for training the EpiCas-DL CNN regressor with 1–5 nt filter groups, MaxPooling, and 80/60/40 fully connected layers)
- **scikit-learn 0.24.2** (Backend for the comparator algorithms (Gradient Boosting, Random Forest) fitted on identical inputs)
- **GPyOpt** (Bayesian hyperparameter optimization of the CNN (30 initial random points, 300 acquisitions))
- **pybedtools 0.9.0** (Custom Python scripting for assembling sgRNA coordinate and epigenetic feature inputs)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Spearman correlation between predicted and observed sgRNA activity on held-out test folds matches the published CRISPRoff_tiling result within reasonable variance (Fig. 2b)
- MSE and cosine similarity for EpiCas-DL on CRISPRoff_tiling are the best among all five models (Fig. 2c)
- Student's t-test across 10 folds shows EpiCas-DL regression performance is significantly better than Random Forest, Gradient Boosting, DeepHF, and C-RNNcrispr
- Input matrices have the expected 11x40 shape: 4 one-hot sequence channels plus min-max normalized epigenetic channels
- Activity scores reproduce the gene-level normalization: each sgRNA phenotype score divided by the absolute average score of the top 3 sgRNAs for that gene

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression were considered; incomplete epigenetic feature information limited model performance
- Available CRISPRoff screening datasets are relatively small and mainly in HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance when testing across systems
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy

## Evidence

- [methods] CRISPRoff_tiling dataset size and scope: "CRISPRoff_tiling dataset contains 111,682 sgRNAs within ± 2.5 kb from the TSS of 520 genes"
- [methods] Train/test split and cross-validation scheme: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [results] Evaluation metric for regression performance: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] Bayesian hyperparameter optimization setup: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [discussion] Cross-system limitation on prediction performance: "The differences between CRISPRoff and CRISPRi systems might affect the prediction performance of EpiCas-DL in the testing datasets"
- [readme] README usage of pretrained epigenetic model: "python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line"
