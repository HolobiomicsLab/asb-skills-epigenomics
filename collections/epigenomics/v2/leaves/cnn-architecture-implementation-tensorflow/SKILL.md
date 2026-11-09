---
name: cnn-architecture-implementation-tensorflow
description: Use when a task asks to reproduce, extend, or validate the EpiCas-DL
  sgRNA-activity CNN — i.e. one-hot encoded 40-nt sequences plus min–max normalized
  epigenetic tracks must be mapped through a 150-filter multi-size convolutional layer
  in TensorFlow/Keras.
when_to_use_negative:
- The task is classical machine learning on a flat feature table — the paper routes
  that through XGBoost/Gradient Boosting/Random Forest rather than this CNN
- No epigenetic annotations are available for the target cell type, so the 11-channel
  tensor cannot be assembled (the repo instead offers sequence-only models, e.g. CRISPRoff_seq/CRISPRa_seq)
- Input sgRNAs fall outside the 40-nt encoded window or lack TSS/gene coordinates
  needed to build the 11 × 40 feature matrix
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Framework whose CNN topology (150 multi-size filters, MaxPooling, BN + dropout
    0.4, dense 80/60/40) is being re-implemented; also provides example.py for running
    the saved models
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend used to define, compile, and save the CNN architecture as an .h5 model
    file
- name: GPyOpt
  role: Bayesian optimization of CNN hyperparameters, executed with 300 acquisitions
    instead of random grid search
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:a90d49709f449a1c475fd61285b3d3601824937244f170ba8dc6e4c2e8f08ec5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python 3.6
  - TensorFlow
  - TensorFlow 2.4.1
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# cnn-architecture-implementation-tensorflow

## Summary

Re-implements the EpiCas-DL 1D CNN in TensorFlow 2.4.1: an 11 × 40 encoded sgRNA tensor (sequence + epigenetic channels) passes through a single convolutional layer of 150 multi-size filters (30 each of 1–5 nt), MaxPooling, batch normalization + dropout 0.4, and dense layers of 80/60/40 units with a linear output for predicting sgRNA activity.

## When to use

Use when a task asks to reproduce, extend, or validate the EpiCas-DL sgRNA-activity CNN — i.e. one-hot encoded 40-nt sequences plus min–max normalized epigenetic tracks must be mapped through a 150-filter multi-size convolutional layer in TensorFlow/Keras.

## When NOT to use

- The task is classical machine learning on a flat feature table — the paper routes that through XGBoost/Gradient Boosting/Random Forest rather than this CNN
- No epigenetic annotations are available for the target cell type, so the 11-channel tensor cannot be assembled (the repo instead offers sequence-only models, e.g. CRISPRoff_seq/CRISPRa_seq)
- Input sgRNAs fall outside the 40-nt encoded window or lack TSS/gene coordinates needed to build the 11 × 40 feature matrix

## Inputs

- 40-nt sgRNA sequences one-hot encoded into an image-like 4-channel scheme
- min–max normalized epigenetic tracks: 4 nucleosome positioning channels, chromatin accessibility (ATACdb ATAC-seq), DNA methylation, and gene expression
- activity-score-normalized screening datasets (CRISPRoff, CRISPRi, CRISPRa) with per-sgRNA normalized phenotype scores

## Outputs

- Compiled Keras/TensorFlow 2.4.1 CNN saved as epicas_dl_cnn_model.h5
- Model summary confirming 150 multi-size convolutional filters, MaxPooling, batch normalization + dropout 0.4, and dense layers of 80/60/40 units with a linear output
- Predicted sgRNA on-target editing efficiency (continuous activity scores) for epigenome editing

## How to apply

Define an 11 × 40 input tensor matching the encoded sgRNA feature matrix: 4 one-hot sequence channels (image-like coding of the 40-nt protospacer), 4 nucleosome positioning channels, 1 chromatin accessibility, 1 DNA methylation, and 1 gene expression channel, with all epigenetic features min–max normalized before model input. Add a single convolutional layer of 150 parallel filters — 30 each of sizes 1, 2, 3, 4, and 5 nucleotides — with ReLU activation, then apply MaxPooling, batch normalization, and dropout with rate 0.4 to prevent overfitting. Stack three fully connected ReLU layers of 80, 60, and 40 units and terminate with a linear-activation output layer that emits continuous sgRNA activity scores. Compile the model with TensorFlow 2.4.1 as the backend (scikit-learn 0.24.2 for classical ML comparisons) and save the compiled architecture to epicas_dl_cnn_model.h5. Tune hyperparameters as in the paper using GPyOpt Bayesian optimization (~300 acquisitions) under a 9:1 train/test split with 5-fold cross-validation.

## Related tools

- **EpiCas-DL** (Framework whose CNN topology (150 multi-size filters, MaxPooling, BN + dropout 0.4, dense 80/60/40) is being re-implemented; also provides example.py for running the saved models) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend used to define, compile, and save the CNN architecture as an .h5 model file)
- **GPyOpt** (Bayesian optimization of CNN hyperparameters, executed with 300 acquisitions instead of random grid search)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- The saved .h5 file loads in TensorFlow 2.4.1 and model.summary() shows one convolutional layer totaling 150 filters across five kernel sizes (30 each of 1, 2, 3, 4, and 5 nt)
- The architecture contains MaxPooling followed by batch normalization and dropout with rate 0.4
- The dense stack is exactly 80, 60, and 40 ReLU units terminated by a linear-activation output layer
- The input shape is 11 × 40 (4 sequence + 4 nucleosome positioning + 1 chromatin accessibility + 1 DNA methylation + 1 gene expression channels) and training proceeds without shape errors
- Epigenetic channels are min–max normalized and the 40-nt sequences are one-hot encoded before fitting, matching the paper's input preparation

## Limitations

- Small CRISPRoff screening datasets, mainly in HEK293T and K562 cell lines, increase overfitting risk and limit generalization (mitigated by dropout 0.4)
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are considered; incomplete epigenetic feature information limited model performance
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance when testing on other datasets

## Evidence

- [methods] Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [methods] we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [readme] EpiCas-DL is a deeplearning based prediction tool for the sgRNA editing effects of CRISPR/dCas editing system.: "EpiCas-DL is a deeplearning based prediction tool for the sgRNA editing effects of CRISPR/dCas editing system."
- [methods] Add a single convolutional layer with five parallel filter sizes — 30 1-nt, 30 2-nt, 30 3-nt, 30 4-nt, and 30 5-nt filters (150 total) — using ReLU activation.: [evidence span withheld — anchor status: not_found]
