---
name: deep-neural-network-training-keras
description: Use when you must train a supervised deep neural network on annotated
  variant features (e.g. EIGEN/CADD/DANN/GWAVA/FATHMM scores, GNOMAD frequency, brain
  eQTL and enhancer/promoter flags) to output a 0–1 prioritization score for non-coding
  variants.
when_to_use_negative:
- The target is step 2 gene prioritization (iMEGES gene score), which uses a different
  deep learning integration of the ncDeepBrain score with general (RVIS, GTEx, haploinsufficiency)
  and disease-specific gene scores — not this variant-level training.
- Input is only raw sequence/reads with no per-variant annotation scores from EIGEN,
  CADD, DANN, GWAVA, FATHMM, GNOMAD, brain eQTLs, and enhancer/promoter tracks.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Keras
  role: High-level API used to define the multilayer network with 4 hidden layers,
    tuned dropout, and sigmoid output layer
  repo: https://keras.io/
- name: TensorFlow
  role: Backend on which the Keras ncDeepBrain network is trained by SGD with momentum
    and binary cross-entropy
  repo: https://www.tensorflow.org/
- name: ANNOVAR
  role: Annotates dsQTL training variants to assemble the input feature matrix
  repo: http://annovar.openbioinformatics.org/en/latest/
- name: bPCA
  role: R code that imputes missing DANN, CADD, EIGEN, GWAVA and FATHMM feature values
- name: ROSE
  role: R library used to balance the 574-positive vs 27,735-negative class imbalance
    before training
- name: python scikit
  role: Generates ROC curves and AUC during ten-fold cross-validation
  repo: http://scikit-learn.org/stable/documentation.html
- name: CADD
  role: One of the integrated non-coding variant predictor features
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: One of the integrated non-coding variant predictor features
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: One of the integrated non-coding variant predictor features
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM
  role: One of the integrated non-coding variant predictor features
  repo: http://fathmm.biocompute.org.uk/
- name: GNOMAD
  role: Population allele frequency feature for each variant
  repo: http://gnomad.broadinstitute.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:da5f546c30b4e284646cb6032f72b1de5f4f05e74e5aad4b95aa89f09b02367f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANNOVAR
  - CADD
  - DANN
  - FATHMM
  - FATHMM-MKL
  - GNOMAD
  - GWAVA
  - Keras
  - ROSE
  - TensorFlow
  - bPCA
  - deltaSVM
  - python scikit
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# deep-neural-network-training-keras

## Summary

Train a Keras model with TensorFlow backend to produce the ncDeepBrain whole-genome non-coding variant score, integrating five predictor scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, brain eQTLs, and enhancer/promoter indicators into a 0–1 variant prioritization score for brain disorders.

## When to use

Use when you must train a supervised deep neural network on annotated variant features (e.g. EIGEN/CADD/DANN/GWAVA/FATHMM scores, GNOMAD frequency, brain eQTL and enhancer/promoter flags) to output a 0–1 prioritization score for non-coding variants.

## When NOT to use

- The target is step 2 gene prioritization (iMEGES gene score), which uses a different deep learning integration of the ncDeepBrain score with general (RVIS, GTEx, haploinsufficiency) and disease-specific gene scores — not this variant-level training.
- Input is only raw sequence/reads with no per-variant annotation scores from EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, brain eQTLs, and enhancer/promoter tracks.

## Inputs

- dsQTL training variant set: 574 positive and 27,735 negative SNPs (MAF > 5%) annotated with ANNOVAR
- 11-feature matrix: EIGEN, CADD, DANN, GWAVA, FATHMM scores, GNOMAD frequency, brain eQTL score (default 0.5 when no known brain eQTL from CommonMind), binary enhancer/promoter indicators for H3K4me3, H3K4me1, H3K27me3, H3K27ac regions from PsychENCODE and Roadmap Epigenomics brain tissues
- bPCA R code for imputation (passed via -B/--bPCA in iMEGES)

## Outputs

- ncDeepBrain variant score in the 0–1 range from the sigmoid output layer
- ROC curves (sensitivity vs specificity) and AUC per cross-validation fold from python scikit

## How to apply

Annotate the dsQTL training variants (574 positive, 27,735 negative SNPs with MAF > 5%) with ANNOVAR and assemble an 11-feature matrix; impute missing DANN, CADD, EIGEN, GWAVA and FATHMM values with bPCA fitted on non-missing genome-wide scores; balance the imbalanced classes with the ROSE R library. Build a Keras model on the TensorFlow backend with a multilayer network (4 hidden layers, manually tuned node counts and dropout rates) ending in a sigmoid output layer that scales predictions to 0–1. Minimize binary cross-entropy (negative log-likelihood) with stochastic gradient descent with momentum and standard back-propagation. Evaluate with ten-fold cross-validation, generating ROC curves (sensitivity vs specificity) with python scikit; the reference ncDeepBrain model reached ~80% classification AUC on the deltaSVM training dataset 1.

## Related tools

- **Keras** (High-level API used to define the multilayer network with 4 hidden layers, tuned dropout, and sigmoid output layer) — https://keras.io/
- **TensorFlow** (Backend on which the Keras ncDeepBrain network is trained by SGD with momentum and binary cross-entropy) — https://www.tensorflow.org/
- **ANNOVAR** (Annotates dsQTL training variants to assemble the input feature matrix) — http://annovar.openbioinformatics.org/en/latest/
- **bPCA** (R code that imputes missing DANN, CADD, EIGEN, GWAVA and FATHMM feature values)
- **ROSE** (R library used to balance the 574-positive vs 27,735-negative class imbalance before training)
- **python scikit** (Generates ROC curves and AUC during ten-fold cross-validation) — http://scikit-learn.org/stable/documentation.html
- **CADD** (One of the integrated non-coding variant predictor features) — http://cadd.gs.washington.edu/
- **DANN** (One of the integrated non-coding variant predictor features) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (One of the integrated non-coding variant predictor features) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM** (One of the integrated non-coding variant predictor features) — http://fathmm.biocompute.org.uk/
- **GNOMAD** (Population allele frequency feature for each variant) — http://gnomad.broadinstitute.org/

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o ncDeepBrain_out.txt -buildver hg19
```

## Evaluation signals

- Ten-fold cross-validation ROC/AUC: classification AUC should approximate the reported 80% on the deltaSVM data training dataset 1 (Additional file 1: Figure S2(a)).
- Sigmoid output layer restricts all ncDeepBrain scores to the 0–1 range.
- Feature matrix completeness: no missing values remain after bPCA imputation of DANN, CADD, EIGEN, GWAVA and FATHMM.
- Class balance: training set after ROSE oversampling no longer shows the 574 vs 27,735 positive/negative skew.
- Loss trajectory: training minimizes binary cross-entropy without divergence under SGD with momentum.

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models.
- Some procedures rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal.
- Node counts and dropout rates were manually tuned rather than systematically optimized.

## Evidence

- [results] ncDeepBrain achieved classification AUC of 80% on training dataset 1 (deltaSVM data): "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [intro] Step 1 deep learning to build the whole genome variant score: "we used a deep learning approach to build a whole genome variant score for variants which affect brain functions"
- [readme] Integrated predictor scores feeding the deep learning model: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
- [readme] Keras with TensorFlow backend: "We used keras library with Tensorflow backend for deep learning in our study."
