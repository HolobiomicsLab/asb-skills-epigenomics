---
name: cnn-sgrna-activity-prediction
description: Use when predicting sgRNA editing/knock-down or knock-up efficiency from
  a 40-nt target sequence plus genomic coordinates and epigenetic tracks (methylation,
  ATAC accessibility, RNA expression, nucleosome/TSS distance), e.g., screening dataset
  scoring or sgRNA design ranking.
when_to_use_negative:
- The input is already a ranked sgRNA feature table and no sequence-to-activity mapping
  is needed — use a comparison/evaluation skill instead
- Only a single epigenetic feature is available and the goal is ablation attribution
  rather than best-effort prediction — the all-features model is the validated configuration
- The target system is a nuclease-based CRISPR editing experiment rather than dCas-based
  epigenome editing (CRISPRoff/CRISPRi/CRISPRa), for which the models were trained
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: CNN framework providing seq-only and seq+epigenetic pretrained models for
    sgRNA activity prediction
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend for building and running the EpiCas-DL convolutional neural network
    models
- name: scikit-learn 0.24.2
  role: Backend for machine-learning baselines and computing ROC-AUC / Spearman correlation
    evaluation metrics
- name: GPyOpt
  role: Bayesian hyperparameter optimization of the CNN (30 initial random points,
    300 acquisitions)
- name: pybedtools
  role: Custom Python scripting for intersecting sgRNA coordinates with epigenetic
    tracks
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:20b5620eb9bc271591f3050e60dc8f33163df92dbc98462721af6c1a0f1723a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python
  - Python 3.6
  - TensorFlow 2.4.1
  - pybedtools
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# cnn-sgrna-activity-prediction

## Summary

Predict sgRNA on-target activity for CRISPR-mediated epigenome editing (CRISPRoff/CRISPRi silencing, CRISPRa activation) using the EpiCas-DL convolutional neural network, which fuses one-hot sequence encoding with min-max normalized epigenetic channels (nucleosome positioning/TSS distance, chromatin accessibility, DNA methylation, RNA expression).

## When to use

Use when predicting sgRNA editing/knock-down or knock-up efficiency from a 40-nt target sequence plus genomic coordinates and epigenetic tracks (methylation, ATAC accessibility, RNA expression, nucleosome/TSS distance), e.g., screening dataset scoring or sgRNA design ranking.

## When NOT to use

- The input is already a ranked sgRNA feature table and no sequence-to-activity mapping is needed — use a comparison/evaluation skill instead
- Only a single epigenetic feature is available and the goal is ablation attribution rather than best-effort prediction — the all-features model is the validated configuration
- The target system is a nuclease-based CRISPR editing experiment rather than dCas-based epigenome editing (CRISPRoff/CRISPRi/CRISPRa), for which the models were trained

## Inputs

- sgRNA target sequence (40 nt, e.g. GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC)
- Genomic coordinates of the target site (chromosome, start, end, PAM position, strand/transcript, gene, cell line)
- Epigenetic reference tracks (nucleosome positioning, chromatin accessibility from ATACdb, DNA methylation, RNA expression) from the epi_reference folder
- CRISPRoff/CRISPRi/CRISPRa screening datasets with phenotype or activity scores (e.g. CRISPRoff_tiling with 111,682 sgRNAs on 520 genes)

## Outputs

- Per-sgRNA predicted activity score (regression output) or binary high/low-activity label (classification output)
- Model performance metrics: ROC-AUC and Spearman correlation with s.e.m. over cross-validation folds
- Trained/pretrained model checkpoints (CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi)

## How to apply

Encode each sgRNA target site as an 11 x 40 matrix: four one-hot sequence channels (one-hot encoding strategy formulating the sequence into an image-like coding scheme) plus epigenetic channels min-max normalized before model input. Split data 90% training / 10% testing and run 5-fold cross-validation on the training set, tuning the CNN architecture (five convolution filter sizes, 30 filters each of 1-5 nt; three fully connected layers of 80/60/40 units; dropout 0.4 after batch normalization; linear output) with GPyOpt Bayesian optimization (30 initial random points, 300 acquisitions). Build with TensorFlow 2.4.1 and scikit-learn 0.24.2 backends; classify sgRNAs as high-activity using phenotype scores below -0.1 (or activity scores above 0.75 for activityscore datasets) and evaluate with ROC-AUC (classification) and Spearman correlation (regression). Use the pretrained CRISPRoff_seq/CRISPRoff_epi or CRISPRa_seq/CRISPRa_epi models from the repository for inference via example.py, passing chrom, start, end, pamCoord, transcript, gene and cell_line so the epigenetic channels can be extracted from the epi_reference tracks.

## Related tools

- **EpiCas-DL** (CNN framework providing seq-only and seq+epigenetic pretrained models for sgRNA activity prediction) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend for building and running the EpiCas-DL convolutional neural network models)
- **scikit-learn 0.24.2** (Backend for machine-learning baselines and computing ROC-AUC / Spearman correlation evaluation metrics)
- **GPyOpt** (Bayesian hyperparameter optimization of the CNN (30 initial random points, 300 acquisitions))
- **pybedtools** (Custom Python scripting for intersecting sgRNA coordinates with epigenetic tracks) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Input matrices are exactly 11 x 40 (4 sequence + 7 epigenetic channels by 40 positions) with sequence channels one-hot and epigenetic channels min-max scaled to [0,1]
- ROC-AUC for classification falls in [0, 1] and reproduced independent-validation AUC is around the reported 0.746 on combined independent datasets
- The all-features (seq + all epigenetic channels) model attains the highest AUC and Spearman correlation versus sequence-only and seq-plus-single-feature ablations, matching the Fig. 1c-d ordering
- Independent-validation metrics file contains one entry per dataset (CRISPRi_activityscore, CRISPRi_K562, CRISPRoff_genome, hCRISPRi-v2, CRISPRi_genome, CRISPRa_activityscore) with both ROC-AUC and Spearman columns
- Ablation summary table (feature_ablation_results.csv) contains one row per feature combination including the all-features condition

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Available CRISPRoff screening datasets were small and mainly in HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance when testing on the independent datasets
- Screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy

## Evidence

- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [methods] 9:1 split with 5-fold CV: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] TensorFlow and scikit-learn backends: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [readme] README model description: "EpiCas-DL is a deeplearning based prediction tool for the sgRNA editing effects of CRISPR/dCas editing system"
- [discussion] Interactive website: "we also provide an interactive website for rapid and convenient exploration at http://www.sunlab.fun:3838/EpiCas-DL"
