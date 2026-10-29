---
name: cnn-regression-model-training
description: 'Use when predicting continuous sgRNA gene-repression/activation activity
  from encoded sequence-plus-epigenetic matrices (11x40: 4 one-hot channels + nucleosome,
  chromatin accessibility, methylation, expression), or reproducing Fig. 2b-2c benchmark.'
when_to_use_negative:
- The task is classification of on-target editing outcomes (the paper instead computes
  AUC for classification and Spearman correlation for regression — use the classification
  variant of the framework)
- Only raw sgRNA sequences are available with no epigenetic annotations and no access
  to the 'epi_reference' data needed to build the methylation/accessibility/expression
  channels
- The dataset is not epigenome-editing (CRISPRoff/CRISPRi/CRISPRa) screening data;
  nuclease Cas9 efficiency datasets fall outside this model's scope
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: EpiCas-DL
  role: 'The CNN regression framework being trained: five 1-5 nt filter groups, 80/60/40
    FC layers, dropout 0.4, linear output'
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for building and training the CNN regression model
- name: GPyOpt
  role: Bayesian hyperparameter optimization (30 initial random points, 300 acquisitions)
    of the CNN architecture
- name: scikit-learn 0.24.2
  role: Backend for the Gradient Boosting and Random Forest comparator algorithms
    on identical inputs
- name: Python 3.6
  role: Language for custom data-preparation scripts with pybedtools to build the
    feature matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:3bb8ec6405d6601942e92e1286889c13dc9e09d010b8085129588727a8aca2f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python 3.6
  - TensorFlow 2.4.1
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# CNN regression model training

## Summary

Trains a 1D convolutional neural network regressor (EpiCas-DL style) that combines one-hot encoded sgRNA sequence channels with epigenetic feature channels to predict continuous sgRNA activity scores, and benchmarks it against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr baselines.

## When to use

Use when predicting continuous sgRNA gene-repression/activation activity from encoded sequence-plus-epigenetic matrices (11x40: 4 one-hot channels + nucleosome, chromatin accessibility, methylation, expression), or reproducing Fig. 2b-2c benchmark.

## When NOT to use

- The task is classification of on-target editing outcomes (the paper instead computes AUC for classification and Spearman correlation for regression — use the classification variant of the framework)
- Only raw sgRNA sequences are available with no epigenetic annotations and no access to the 'epi_reference' data needed to build the methylation/accessibility/expression channels
- The dataset is not epigenome-editing (CRISPRoff/CRISPRi/CRISPRa) screening data; nuclease Cas9 efficiency datasets fall outside this model's scope

## Inputs

- CRISPRoff_tiling sgRNA table (111,682 sgRNAs within ±2.5 kb of the TSS of 520 genes) with gene-level normalized activity scores
- 11x40 encoded feature matrices per sgRNA (4 one-hot sequence channels + nucleosome positioning, chromatin accessibility, DNA methylation, gene expression channels)
- Trained model files from the repository, e.g. CRISPRoff_seq / CRISPRoff_epi / CRISPRa_seq / CRISPRa_epi

## Outputs

- Trained TensorFlow CNN regressor (linear output) for sgRNA activity prediction
- Per-fold Spearman correlation, mean squared error, and cosine similarity for EpiCas-DL and the four baselines across 10 folds
- Student's t-test comparison of EpiCas-DL vs. Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr

## How to apply

Encode each sgRNA target site as an 11x40 image-like matrix (4 one-hot sequence channels plus min-max normalized nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression channels) and normalize phenotype scores to gene-level activity scores using the absolute average of the top 3 sgRNAs per gene. Split data 9:1 into train/test and run cross-validation on the training portion, tuning the architecture with GPyOpt Bayesian optimization (30 initial random points, 300 acquisitions) rather than grid search. Build the CNN in TensorFlow 2.4.1 with five convolutional filter groups (30 filters each, filter lengths 1-5 nt), MaxPooling, three fully connected layers (80/60/40 units), batch normalization followed by dropout 0.4, ReLU activations, and a linear output layer for regression. Fit the four comparators (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) via scikit-learn 0.24.2 on identical inputs with default configurations. Evaluate across the 10 folds by Spearman correlation, MSE, and cosine similarity on the held-out folds and test EpiCas-DL vs. baselines with Student's t-test.

## Related tools

- **EpiCas-DL** (The CNN regression framework being trained: five 1-5 nt filter groups, 80/60/40 FC layers, dropout 0.4, linear output) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for building and training the CNN regression model)
- **GPyOpt** (Bayesian hyperparameter optimization (30 initial random points, 300 acquisitions) of the CNN architecture)
- **scikit-learn 0.24.2** (Backend for the Gradient Boosting and Random Forest comparator algorithms on identical inputs)
- **Python 3.6** (Language for custom data-preparation scripts with pybedtools to build the feature matrices)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Reported Spearman correlation, MSE, and cosine similarity for EpiCas-DL on CRISPRoff_tiling match Fig. 2b-2c values within reasonable variance across the 10 folds
- Student's t-test shows EpiCas-DL regression performance is significantly better than Random Forest, Gradient Boosting, DeepHF, and C-RNNcrispr
- Input matrices have the expected 11x40 shape (4 one-hot sequence channels + 7 epigenetic/derived channels) with min-max normalized epigenetic features
- Activity scores are gene-level normalized using the absolute average of the top 3 sgRNAs per gene before training
- Hyperparameter search log shows GPyOpt with 30 initial random points and 300 acquisitions completed

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Available CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, increasing overfitting risk and limiting generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance on testing datasets
- Screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy

## Evidence

- [methods] CNN architecture with five filter groups and dropout: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [results] Train/test split and evaluation metric: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] Comparator algorithms on identical inputs: "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
- [methods] Deep learning backend: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [readme] Combined sequence and epigenetic feature encoding: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
- [readme] Available trained model checkpoints: "1.There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
