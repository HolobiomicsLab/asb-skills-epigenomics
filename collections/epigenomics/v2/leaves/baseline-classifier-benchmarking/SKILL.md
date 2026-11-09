---
name: baseline-classifier-benchmarking
description: 'Use when a new sgRNA-activity prediction model must be shown competitive
  with established methods: you have binarized sgRNA labels and need a like-for-like
  ROC-AUC comparison table against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr.'
when_to_use_negative:
- The task is regression/efficiency prediction of continuous sgRNA activity scores
  rather than binary classification (highly active vs. not) evaluated by ROC-AUC
- The target epigenetic features (nucleosome positioning, ATAC-seq accessibility,
  DNA methylation, RNA expression) are unavailable for the cell line of interest,
  making the sequence-plus-epigenetic input matrix unbuildable for a like-for-like
  comparison
- The dataset is not an sgRNA screening dataset with phenotype scores to binarize
  at the −0.1 cutoff (e.g., already aggregated per-gene summaries)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: Deep-learning CNN classifier under benchmark; trained with 5-fold CV within
    the 90% training split and GPyOpt Bayesian hyperparameter optimization
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for training the EpiCas-DL deep learning model
- name: scikit-learn 0.24.2
  role: Machine-learning backend for the Gradient Boosting and Random Forest baselines
- name: GPyOpt
  role: Bayesian optimization package for tuning EpiCas-DL hyperparameters (30 initial
    random points, 300 acquisitions)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:77a7b3fad6f8d1ea6767482ef09e56eea79be917ad7a9dfe645acae73356e1ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - TensorFlow 2.4.1
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# baseline-classifier-benchmarking

## Summary

Benchmark a deep learning sgRNA-activity classifier (EpiCas-DL) against shallow and published deep-learning baselines (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) using ROC-AUC under 10-fold cross-validation on an identical sequence-plus-epigenetic input matrix.

## When to use

Use when a new sgRNA-activity prediction model must be shown competitive with established methods: you have binarized sgRNA labels and need a like-for-like ROC-AUC comparison table against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr.

## When NOT to use

- The task is regression/efficiency prediction of continuous sgRNA activity scores rather than binary classification (highly active vs. not) evaluated by ROC-AUC
- The target epigenetic features (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression) are unavailable for the cell line of interest, making the sequence-plus-epigenetic input matrix unbuildable for a like-for-like comparison
- The dataset is not an sgRNA screening dataset with phenotype scores to binarize at the −0.1 cutoff (e.g., already aggregated per-gene summaries)

## Inputs

- CRISPRoff_tiling sgRNA dataset (111,682 sgRNAs within ± 2.5 kb of the TSS of 520 genes) with phenotype scores
- 11 × 40 per-sgRNA input matrices: one-hot encoded 40 bp sequence (4 channels) plus min–max normalized epigenetic channels (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression)
- Binarized activity labels (phenotype score < −0.1 → highly active '1')

## Outputs

- auc_comparison_table.csv with ROC-AUC entries for EpiCas-DL, DeepHF, C-RNNcrispr, Random Forest, and Gradient Boosting (mean ± s.e.m. over 10 CV folds)
- Per-fold cross-validation AUC scores for each model on the held-out 10% test set

## How to apply

Assemble an identical 11 × 40 input matrix for every model — one-hot encoded 40 bp sgRNA sequence (4 channels) plus min–max normalized epigenetic channels (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression) — so that performance differences reflect model architecture rather than feature availability. Randomly split the data 9:1 into training (90%) and testing (10%) sets, fitting EpiCas-DL (CNN with five filter groups of 30 filters at 1–5 nt widths, MaxPooling, three fully connected layers of 80/60/40 units, dropout 0.4 after batch normalization, GPyOpt Bayesian hyperparameter optimization with 30 initial random points and 300 acquisitions) in TensorFlow 2.4.1, and the baselines with scikit-learn 0.24.2 as the machine-learning backend using default configurations. Evaluate all five models on the held-out test set by computing ROC-AUC under 10-fold cross-validation and report per-fold means with s.e.m.; binarize sgRNA labels with the published cutoff (phenotype scores lower than −0.1 labeled as highly active '1'). Assemble the five-model comparison table and judge correctness by checking baseline AUCs against the reported values (DeepHF 0.751, C-RNNcrispr 0.790, Random Forest 0.767, Gradient Boosting 0.755) with EpiCas-DL at 0.879.

## Related tools

- **EpiCas-DL** (Deep-learning CNN classifier under benchmark; trained with 5-fold CV within the 90% training split and GPyOpt Bayesian hyperparameter optimization) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for training the EpiCas-DL deep learning model)
- **scikit-learn 0.24.2** (Machine-learning backend for the Gradient Boosting and Random Forest baselines)
- **GPyOpt** (Bayesian optimization package for tuning EpiCas-DL hyperparameters (30 initial random points, 300 acquisitions))

## Examples

```
git clone https://github.com/yangqianq/EpiCas-DL.git && python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- auc_comparison_table.csv contains ROC-AUC entries for all five models (EpiCas-DL, DeepHF, C-RNNcrispr, Random Forest, Gradient Boosting)
- Baseline AUCs match reported values: DeepHF 0.751, C-RNNcrispr 0.790, Random Forest 0.767, Gradient Boosting 0.755
- EpiCas-DL achieves a competitive or superior AUC (0.879) on the CRISPRoff_tiling held-out test set
- All models were fitted on the identical sequence-plus-epigenetic input; ranks are consistent with the expected ordering EpiCas-DL > C-RNNcrispr > Random Forest > Gradient Boosting > DeepHF
- Per-fold AUC scores from the 10-fold cross-validation are reported as means with s.e.m.

## Limitations

- Small CRISPRoff screening datasets, mainly in HEK293T and K562 cell lines, may increase overfitting risk and limit generalization
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets

## Evidence

- [results] Randomly divided dataset into 90% training and 10% testing with 10-fold cross-validations: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] Model comparison with Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr: "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
- [methods] Scikit-learn and TensorFlow as backends: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [methods] Bayesian hyperparameter optimization with GPyOpt: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [readme] EpiCas-DL web tool availability from README: "EpiCas-DL is a deeplearning based prediction tool for the sgRNA editing effects of CRISPR/dCas editing system."
