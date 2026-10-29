---
name: gene-prioritization-deep-learning
description: Use when you have non-coding variants already scored by a brain-variant
  prioritizer (ncDeepBrain) and need to assign per-gene susceptibility probabilities
  by linking variants to genes and training a deep network on gene-level features.
when_to_use_negative:
- You need to score individual non-coding variants rather than prioritize genes —
  that is iMEGES step 1 (ncDeepBrain variant prioritization)
- Your regulatory candidates act via distal elements where assigning a variant to
  its closest gene within 100 kb is inappropriate (e.g. enhancers regulating non-nearest
  genes)
- Input is already a per-gene prioritized ranking with no variant-level ncDeepBrain
  scores to integrate
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3053
tools:
- name: iMEGES
  role: Implements the two-step pipeline; step 2 produces the iMEGES gene score per
    mutated gene
  repo: https://github.com/WGLab/iMEGES
- name: TensorFlow
  role: Backend for the Keras deep neural network used in gene prioritization
  repo: https://github.com/WGLab/iMEGES
- name: GTEx
  role: Provides gene/variant expression-based q-values (q <= 0.05 across 44 tissues)
    as a general gene score feature
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:c54d0af006980ec05f021ef7a9aba948f60723bdaed30983725a02d509420da5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - GTEx
  - TensorFlow
  - iMEGES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-prioritization-deep-learning

## Summary

Second step of iMEGES: a Keras/TensorFlow multilayer neural network integrates the ncDeepBrain non-coding variant score with general gene scores (RVIS, GTEx, haploinsufficiency) and disease-specific scores (Phenolyzer, CNV, de novo mutation) to output an iMEGES gene score — a susceptibility probability per mutated gene for mental disorders.

## When to use

Use when you have non-coding variants already scored by a brain-variant prioritizer (ncDeepBrain) and need to assign per-gene susceptibility probabilities by linking variants to genes and training a deep network on gene-level features.

## When NOT to use

- You need to score individual non-coding variants rather than prioritize genes — that is iMEGES step 1 (ncDeepBrain variant prioritization)
- Your regulatory candidates act via distal elements where assigning a variant to its closest gene within 100 kb is inappropriate (e.g. enhancers regulating non-nearest genes)
- Input is already a per-gene prioritized ranking with no variant-level ncDeepBrain scores to integrate

## Inputs

- Annotated variant list from iMEGES step 1 with ncDeepBrain scores (Annovar/bed format)
- Gene-level annotations: RVIS, GTEx and haploinsufficiency scores
- Disease-specific scores: Phenolyzer gene scores, CNV q-values, de novo mutation scores
- Known brain eQTLs from CommonMind (bed file, -E option)
- bPCA imputation R script (-B bPCA.R)
- Patient phenotype and genotype information

## Outputs

- iMEGES gene score (deep learning probability 0-1) per mutated gene, e.g. written to imeges_gene_scores.tsv
- Ranked list of disease-specific prioritized genes
- Ten-fold cross-validation ROC curves and AUC values (figure files)

## How to apply

Build a gene-level feature vector F = (ncDeepBrain, RVIS, GTEx, haploinsufficiency, Phenolyzer, CNV, de novo mutation) for each mutated gene, attaching each non-coding variant's ncDeepBrain score to its closest gene within 100 kb genomic distance. Check summary statistics and pairwise correlations of feature variables for outliers and collinearity before training, then fill missing values with bPCA imputation and apply default fills (e.g. 0.5 for missing CNV q-values, 0 for missing RVIS). Train a Keras network with TensorFlow backend — four hidden layers with dropout regularization and a sigmoid output scaled to 0–1 — minimizing negative log-likelihood loss via stochastic gradient descent with momentum, using ROSE-balanced training data. Predict the deep learning probability (iMEGES gene score) for every mutated gene and rank genes accordingly; evaluate with ten-fold cross-validation, reporting ROC curves and AUC for discriminating disease genes from non-disease genes (reported ~57% schizophrenia, ~58% ASD).

## Related tools

- **iMEGES** (Implements the two-step pipeline; step 2 produces the iMEGES gene score per mutated gene) — https://github.com/WGLab/iMEGES
- **TensorFlow** (Backend for the Keras deep neural network used in gene prioritization) — https://github.com/WGLab/iMEGES
- **GTEx** (Provides gene/variant expression-based q-values (q <= 0.05 across 44 tissues) as a general gene score feature)

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o imeges_gene_scores.tsv -buildver hg19
```

## Evaluation signals

- Ten-fold cross-validation ROC/AUC on held-out disease vs non-disease genes; paper reports ~57% AUC (schizophrenia) and ~58% AUC (ASD) — much lower AUCs suggest feature assembly or training errors
- Gene scores are probabilities in [0,1] from the sigmoid output layer; values outside this range indicate a broken output scaling
- Every non-coding variant should map to a gene only if within 100 kb of the gene; verify the 100 kb assignment rule was applied
- Missing feature values should be imputed via bPCA with default fills (0.5 for missing CNV q-values, 0 for missing RVIS) — count of NaNs after imputation should be zero
- Genes with multiple mutations should have all mutations considered, not only the highest-scoring one

## Limitations

- Each non-coding variant is linked to its closest gene, which the authors note may work well for promoters but is less optimal for other regulatory elements
- Some disease-specific scores derive from imputed GWAS data, where hits may be proxy markers rather than causal variants, making the model less reliable than ideal
- Large-scale high-quality training data for deep learning models remain difficult to obtain

## Evidence

- [results] Gene prioritization of iMEGES used a deep learning framework to integrate the ncDeepBrain score, general scores ... and the disease specific scores: [evidence span withheld — anchor status: not_found]
- [results] First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization.: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
- [readme] To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less.: "To link the non-coding variants to a gene, we assigned each of these variants to it closet gene in term of genome distance, i.e., distance to gene is 100 KB or less."
- [readme] After integration of these pieces of information, the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score: "After integration of these pieces of information, the output of the second Step is the deep learning probability for each of the mutated gene and call it the iMEGES gene score"
- [discussion] we associated each non-coding variant to its closest gene in the genome. This strategy may work well for promotors: "we associated each non-coding variant to its closest gene in the genome. This strategy may work well for promotors"
