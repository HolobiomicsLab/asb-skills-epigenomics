---
name: neural-network-sigmoid-classifier-training
description: Use when gene-level feature vectors combining a non-coding variant deep-learning
  score with gene-based scores must be fused into a per-gene susceptibility probability
  by a supervised neural network classifier.
when_to_use_negative:
- The task is variant-level (non-coding variant) prioritization — that is step 1 (ncDeepBrain),
  not this gene-prioritization classifier.
- A validated pre-trained gene score already exists and no retraining on disease-gene
  labels is needed.
- Input variants cannot be assigned to genes (no genomic coordinates for the 100 kb
  closest-gene linkage).
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: TensorFlow
  role: Backend for the Keras deep neural network that produces the iMEGES gene score
  repo: https://github.com/WGLab/iMEGES
- name: iMEGES
  role: Pipeline whose step 2 (gene prioritization) implements this classifier
  repo: https://github.com/WGLab/iMEGES
- name: GTEx
  role: Provides gene-based tissue expression scores used as input features
- name: RVIS gene score
  role: Gene-based intolerance score used as an input feature
  repo: http://genic-intolerance.org/
- name: Phenolyzer
  role: Disease-specific gene score derived from patient phenotype terms
  repo: http://phenolyzer.wglab.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:4b89b73b3aef1780dc9d13170315f98c38682c249cb4468c031eb14ce6e84168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - GTEx
  - Phenolyzer
  - RVIS gene score
  - TensorFlow
  - iMEGES
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# neural-network-sigmoid-classifier-training

## Summary

Train a multilayer deep neural network (Keras with TensorFlow backend) that integrates the ncDeepBrain variant score with general gene scores (RVIS, GTEx, haploinsufficiency) and disease-specific scores (Phenolyzer, CNV, de novo mutation) to output a sigmoid-scaled susceptibility probability per mutated gene — the iMEGES gene score.

## When to use

Use when gene-level feature vectors combining a non-coding variant deep-learning score with gene-based scores must be fused into a per-gene susceptibility probability by a supervised neural network classifier.

## When NOT to use

- The task is variant-level (non-coding variant) prioritization — that is step 1 (ncDeepBrain), not this gene-prioritization classifier.
- A validated pre-trained gene score already exists and no retraining on disease-gene labels is needed.
- Input variants cannot be assigned to genes (no genomic coordinates for the 100 kb closest-gene linkage).

## Inputs

- Per-gene feature matrix with columns: ncDeepBrain score, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV q-value, de novo mutation score
- Variant-to-gene assignments (each non-coding variant to its closest gene within 100 kb)
- Balanced training set of disease vs non-disease genes (ROSE-handled), e.g. schizophrenia and ASD gene sets

## Outputs

- iMEGES gene score: a deep learning susceptibility probability (0–1) for each mutated gene
- Ranked gene list with iMEGES gene scores (e.g. imeges_gene_scores.tsv)
- ROC curves and AUC values from ten-fold cross-validation (figure files)

## How to apply

Assemble a gene-level feature vector F = (ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo mutation), attaching each non-coding variant's ncDeepBrain score to its closest gene within 100 kb genomic distance. First examine summary statistics and pairwise correlations of feature variables to detect outliers and collinearity, then fill missing values with bPCA (Bayesian Principal Component Analysis) imputation, applying default fills such as 0.5 for missing CNV q-values and 0 for missing RVIS. Build a multilayer neural network in Keras with a TensorFlow backend using four hidden layers with dropout regularization and a sigmoid output layer scaled to 0–1; minimize the negative log-likelihood loss via stochastic gradient descent with momentum. Train on ROSE-handled balanced training data, then predict the iMEGES gene score for every mutated gene. Evaluate with ten-fold cross-validation, reporting ROC curves and AUC values (≈57% schizophrenia, ≈58% ASD) for discriminating disease genes from non-disease genes.

## Related tools

- **TensorFlow** (Backend for the Keras deep neural network that produces the iMEGES gene score) — https://github.com/WGLab/iMEGES
- **iMEGES** (Pipeline whose step 2 (gene prioritization) implements this classifier) — https://github.com/WGLab/iMEGES
- **GTEx** (Provides gene-based tissue expression scores used as input features)
- **RVIS gene score** (Gene-based intolerance score used as an input feature) — http://genic-intolerance.org/
- **Phenolyzer** (Disease-specific gene score derived from patient phenotype terms) — http://phenolyzer.wglab.org/

## Examples

```
python ncDeepBrain.py --help  # iMEGES CLI; gene-prioritization model built in Keras/TF: keras.Sequential([Dense(units, dropout)... , Dense(1, activation='sigmoid')]) with SGD+momentum, NLL loss
```

## Evaluation signals

- Ten-fold cross-validation ROC/AUC for discriminating disease from non-disease genes should be near reported values (~57% schizophrenia, ~58% ASD).
- Output gene scores must be bounded in [0, 1] (sigmoid output layer).
- Every non-coding variant should be linked to a gene at ≤ 100 kb distance; verify no unassigned variants.
- Feature matrix must pass summary statistics / pairwise correlation checks for outliers and collinearity before training.
- Missing values should be imputed with bPCA and default fills (0.5 for missing CNV q-values, 0 for missing RVIS) — verify no NaNs remain.

## Limitations

- Non-coding variants are assigned to their closest gene, which works well for promoters but may be less optimal for other regulatory elements (e.g. distal enhancers).
- Training depends on large-scale high-quality data, which is challenging to obtain.
- Some disease-specific scores rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal.
- Reported cross-validation AUCs (~57–58%) indicate modest discrimination between disease and non-disease genes.

## Evidence

- [intro] we used another deep learning framework to integrate the ncDeepBrain score, general gene scores (such as GTEx), and disease-specific scores to prioritize mutated genes for mental disorders: "we used another deep learning framework to integrate the ncDeepBrain score, general gene scores (such as GTEx), and disease-specific scores to prioritize mutated genes for mental disorders"
- [readme] To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less.: "To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less."
- [readme] the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score, which measure the susceptibility potential for this gene to be involved in mental disorders.: "the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score, which measure the susceptibility potential for this gene"
- [results] First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization.: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
- [discussion] we associated each non-coding variant to its closest gene in the genome. This strategy may work well for promotors, ... but for other types of regulatory elements, this strategy may be less optimal: [evidence span withheld — anchor status: not_found]
