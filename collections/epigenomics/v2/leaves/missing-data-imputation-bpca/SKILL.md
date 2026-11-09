---
name: missing-data-imputation-bpca
description: Use when iMEGES variant or gene feature matrices contain missing continuous
  predictor scores (e.g. DANN, CADD, EIGEN, GWAVA, FATHMM) that must be filled before
  feeding the deep learning model.
when_to_use_negative:
- The feature matrix is already complete (no missing DANN/CADD/EIGEN/GWAVA/FATHMM
  values) — imputation is unnecessary and could distort true scores
- A feature has a paper-specified default fill rule (missing CNV q-value -> 0.5; missing
  RVIS -> 0) — apply the default instead of bPCA for that column
- The missing entries are the binary enhancer/promoter indicators or the brain eQTL
  column, which are handled by binary encoding or the 0.5 default rather than bPCA
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: bPCA (bPCA.R)
  role: Bayesian PCA implementation used to impute missing variant and gene feature
    scores; supplied in the iMEGES repo and passed via -B/--bPCA
  repo: https://github.com/WGLab/iMEGES
- name: iMEGES / ncDeepBrain.py
  role: Pipeline script that orchestrates annotation, bPCA imputation, and downstream
    deep learning scoring
  repo: https://github.com/WGLab/iMEGES
- name: TensorFlow/Keras
  role: Downstream consumer of the imputed feature matrices; trains the ncDeepBrain
    multilayer network
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:85e9581e455e897c870979c11af7070f57b4f4643caa0a2723832800436fbfed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GTEx
  - GWAVA
  - TensorFlow
  - TensorFlow/Keras
  - bPCA
  - bPCA (bPCA.R)
  - deltaSVM
  - iMEGES
  - iMEGES / ncDeepBrain.py
  - python scikit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# missing-data-imputation-bpca

## Summary

Imputes missing predictor scores in the iMEGES feature matrices (variant-level: DANN, CADD, EIGEN, GWAVA, FATHMM; gene-level features) using Bayesian Principal Component Analysis (bPCA), so the Keras/TensorFlow deep networks can train on complete feature vectors.

## When to use

Use when iMEGES variant or gene feature matrices contain missing continuous predictor scores (e.g. DANN, CADD, EIGEN, GWAVA, FATHMM) that must be filled before feeding the deep learning model.

## When NOT to use

- The feature matrix is already complete (no missing DANN/CADD/EIGEN/GWAVA/FATHMM values) — imputation is unnecessary and could distort true scores
- A feature has a paper-specified default fill rule (missing CNV q-value -> 0.5; missing RVIS -> 0) — apply the default instead of bPCA for that column
- The missing entries are the binary enhancer/promoter indicators or the brain eQTL column, which are handled by binary encoding or the 0.5 default rather than bPCA

## Inputs

- Annotated variant feature matrix (ANNOVAR output) with missing DANN, CADD, EIGEN, GWAVA, FATHMM scores
- Genome-wide non-missing score vectors used to fit the bPCA model
- Gene-level feature vector F = (ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo mutation) with missing entries

## Outputs

- Complete (NA-free) variant feature matrix ready for ROSE balancing and Keras/TensorFlow training
- Complete gene-level feature matrix for the gene-prioritization network
- bPCA.R fit consumed by ncDeepBrain.py via the -B flag

## How to apply

Assemble the 11-feature variant matrix (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTL score with 0.5 default, and binary enhancer/promoter indicators from PsychENCODE/Roadmap Epigenomics H3K4me3, H3K4me1, H3K27me3, H3K27ac regions) or the gene-level feature vector F = (ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo mutation). Fit bPCA on the non-missing genome-wide scores and use the fitted model to impute missing DANN, CADD, EIGEN, GWAVA and FATHMM values; the bPCA R code is supplied in the iMEGES repository and wired into the pipeline via the -B/--bPCA flag of ncDeepBrain.py. Rationale: deep network training requires complete numeric matrices, and bPCA exploits covariance structure among correlated variant scores rather than mean-filling. For gene-level features where the paper specifies defaults, apply the default-fill rules first (0.5 for missing CNV q-values, 0 for missing RVIS) and use bPCA for remaining gaps. Verify the completed matrix has no residual NAs before ROSE class balancing and network training.

## Related tools

- **bPCA (bPCA.R)** (Bayesian PCA implementation used to impute missing variant and gene feature scores; supplied in the iMEGES repo and passed via -B/--bPCA) — https://github.com/WGLab/iMEGES
- **iMEGES / ncDeepBrain.py** (Pipeline script that orchestrates annotation, bPCA imputation, and downstream deep learning scoring) — https://github.com/WGLab/iMEGES
- **TensorFlow/Keras** (Downstream consumer of the imputed feature matrices; trains the ncDeepBrain multilayer network)

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o imeges_out -buildver hg19
```

## Evaluation signals

- After imputation, the variant feature matrix contains zero missing values across the DANN, CADD, EIGEN, GWAVA and FATHMM columns
- The bPCA model was fitted on non-missing genome-wide scores, not on the small dsQTL training set alone
- Downstream ncDeepBrain classification reaches AUC ≈ 80% on the deltaSVM training dataset 1 (Additional file 1: Figure S2(a))
- Gene-level defaults were applied before imputation (0.5 for missing CNV q-values, 0 for missing RVIS) as specified in the paper
- Running ncDeepBrain.py with -B bPCA.R completes without error and the output score file is produced

## Limitations

- Imputed scores are statistical estimates, not true predictor outputs; performance depends on covariance structure among the five variant scores
- The paper notes it is challenging to obtain large-scale high-quality data for training the models, limiting imputation and downstream training reliability
- Some hits rely on imputed GWAS data representing proxy markers rather than causal variants, affecting downstream score interpretability

## Evidence

- [methods] Impute missing values of DANN, CADD, EIGEN, GWAVA and FATHMM features using bPCA (Bayesian Principal Component Analysis) fitted on non-missing genome-wide scores.: [evidence span withheld — anchor status: not_found]
- [methods] Fill missing feature values with bPCA (Bayesian Principal Component Analysis) imputation, and apply the paper's default-fill rules for absent scores (e.g. 0.5 default for missing CNV q-values, 0 for missing RVIS).: [evidence span withheld — anchor status: not_found]
- [readme] The bPCA R code to do the imputation: "-B bPCA.R, --bPCA bPCA.R The bPCA R code to do the imputation"
- [abstract] Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES): "Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES)"
