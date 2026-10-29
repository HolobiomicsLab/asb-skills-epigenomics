---
name: cnn-model-training-with-bayesian-hyperparameter-optimization
description: Use when predicting sgRNA on-target editing efficiency for CRISPR-mediated
  epigenome editing from one-hot encoded 40 bp sequences plus min-max normalized epigenetic
  channels (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression),
  and hyperparameters must be optimized.
when_to_use_negative:
- When the input is already a flat feature table for classical baselines (Random Forest,
  Gradient Boosting) rather than image-like sequence-plus-epigenetic matrices — the
  CNN architecture is designed for the 11 × 40 encoded input
- When screening data are too few or concentrated in a single cell line, since small
  CRISPRoff datasets mainly in HEK293T and K562 increase overfitting risk and limit
  generalization
- When prediction is needed for CRISPRoff data but training used CRISPRi/a data, because
  differences between CRISPRoff and CRISPRi systems can affect test performance
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: CNN deep learning framework being trained (five filter groups 1–5 nt, FC 80/60/40,
    dropout 0.4) for sgRNA activity prediction
  repo: https://github.com/yangqianq/EpiCas-DL
- name: GPyOpt
  role: Bayesian optimization package used to tune CNN hyperparameters (30 initial
    points, 300 acquisitions, 5-fold CV on the training split)
- name: TensorFlow 2.4.1
  role: Backend for building and training the deep learning model
- name: scikit-learn 0.24.2
  role: Backend for training baseline machine-learning algorithms (Random Forest,
    Gradient Boosting) compared against the CNN
- name: pybedtools 0.9.0
  role: Used in custom Python 3.6 scripts for genomic interval processing when assembling
    inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:55707283650c0556449480a36a6abde74881440662e243a7f6f218461d4f1c3b
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# CNN model training with Bayesian hyperparameter optimization

## Summary

Train a 1D CNN (EpiCas-DL) on image-like encoded sgRNA sequence plus epigenetic features (11 × 40 input matrix) in TensorFlow 2.4.1, using GPyOpt Bayesian optimization instead of grid search to tune hyperparameters for sgRNA activity prediction on CRISPRoff/CRISPRi/CRISPRa screening data.

## When to use

Use when predicting sgRNA on-target editing efficiency for CRISPR-mediated epigenome editing from one-hot encoded 40 bp sequences plus min-max normalized epigenetic channels (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression), and hyperparameters must be optimized.

## When NOT to use

- When the input is already a flat feature table for classical baselines (Random Forest, Gradient Boosting) rather than image-like sequence-plus-epigenetic matrices — the CNN architecture is designed for the 11 × 40 encoded input
- When screening data are too few or concentrated in a single cell line, since small CRISPRoff datasets mainly in HEK293T and K562 increase overfitting risk and limit generalization
- When prediction is needed for CRISPRoff data but training used CRISPRi/a data, because differences between CRISPRoff and CRISPRi systems can affect test performance

## Inputs

- sgRNA sequences (40 bp) from CRISPRoff/CRISPRi/CRISPRa tiling screens, e.g. CRISPRoff_tiling (111,682 sgRNAs within ± 2.5 kb of the TSS of 520 genes)
- Phenotype scores per sgRNA for binarizing active/inactive labels (published cutoff: phenotype score < −0.1 labeled as highly active '1')
- Epigenetic feature tracks: nucleosome positioning, ATAC-seq accessibility (from ATACdb), DNA methylation, RNA expression

## Outputs

- Trained EpiCas-DL classification or regression model (four model variants: CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi)
- Per-sgRNA activity predictions and ROC-AUC / Spearman correlation scores on a held-out 10% test set under 10-fold cross-validation
- Optimized hyperparameters from the GPyOpt search (30 initial points, 300 acquisitions)

## How to apply

Formulate each sgRNA as an 11 × 40 image-like matrix: 4 channels of one-hot encoded 40 bp sequence plus 7 min-max normalized epigenetic feature channels. Randomly split the dataset (e.g., CRISPRoff_tiling with 111,682 sgRNAs on 520 genes) 9:1 into training and testing sets, and run 5-fold cross-validation on the training split inside the optimization loop. Optimize hyperparameters with GPyOpt Bayesian optimization using 30 initial random design points followed by 300 acquisitions. Train the EpiCas-DL CNN — five filter groups of 30 filters at 1–5 nt widths, MaxPooling, three fully connected layers of 80/60/40 units, dropout 0.4 after batch normalization, ReLU activations, linear output — in TensorFlow 2.4.1, and evaluate on the held-out 10% with ROC-AUC (classification) or Spearman correlation under 10-fold cross-validation.

## Related tools

- **EpiCas-DL** (CNN deep learning framework being trained (five filter groups 1–5 nt, FC 80/60/40, dropout 0.4) for sgRNA activity prediction) — https://github.com/yangqianq/EpiCas-DL
- **GPyOpt** (Bayesian optimization package used to tune CNN hyperparameters (30 initial points, 300 acquisitions, 5-fold CV on the training split))
- **TensorFlow 2.4.1** (Backend for building and training the deep learning model)
- **scikit-learn 0.24.2** (Backend for training baseline machine-learning algorithms (Random Forest, Gradient Boosting) compared against the CNN)
- **pybedtools 0.9.0** (Used in custom Python 3.6 scripts for genomic interval processing when assembling inputs)

## Examples

```
git clone https://github.com/yangqianq/EpiCas-DL.git; python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096
```

## Evaluation signals

- On CRISPRoff_tiling with 9:1 split, classification model should reach ROC-AUC ≈ 0.879, above DeepHF (0.751), C-RNNcrispr (0.790), Random Forest (0.767), and Gradient Boosting (0.755)
- All epigenetic channels must be min–max normalized and the sgRNA sequence one-hot encoded into 4 channels before model input (11 × 40 matrix)
- GPyOpt run must include 30 initial design points followed by 300 acquisitions with 5-fold cross-validation on the 90% training split
- Regression variant should report Spearman correlation on the held-out 10% test set under 10-fold cross-validation, averaged with s.e.m.
- Repository code should run via `python example.py` with the released seq/epi models (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi)

## Limitations

- Small CRISPRoff screening datasets, mainly in HEK293T and K562 cell lines, may increase overfitting risk in the training process and limit generalization
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance when testing on other datasets
- More complex deep learning frameworks (RNNs, GANs, RL) were not yet explored for improving prediction performance

## Evidence

- [methods] 9:1 split and 5-fold CV: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] GPyOpt optimization details: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [methods] Input encoding: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] Baseline comparison set: "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
- [readme] Backend stack: "tensorflow 2.4.1
         numpy 1.20.1
         pandas 1.2.4
         pybedtools 0.8.2
         GPyOpt 1.2.6"
