---
name: dirichlet-hyperparameter-configuration
description: Use when a cisTopic object holds a binary accessibility matrix (e.g.
  ~97k regulatory regions over 5,335 PBMCs) and multiple LDA models across a topic-number
  grid must be fitted for downstream model selection.
when_to_use_negative:
- cisTopic is deprecated — for new analyses prefer pycisTopic, per the repository
  README
- The input is raw fragment/peak files rather than a binarized accessibility matrix
  already stored in a cisTopic object — first build the object with createcisTopicObjectFrom10X
tools:
- name: cisTopic
  role: R/Bioconductor package providing runModels() for LDA topic-model fitting with
    collapsed Gibbs sampling over the binary accessibility matrix
  repo: http://github.com/aertslab/cisTopic
- name: R
  role: Language in which cisTopic is implemented and the model-fitting workflow is
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:1e1e7ad191f4a4f1b600f17e6c0e47dcf173d125500d98e16f2f7e50dad15476
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dirichlet-hyperparameter-configuration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Configure and fit Latent Dirichlet Allocation (LDA) topic models over a binary accessibility matrix in cisTopic, keeping the default Dirichlet hyperparameters (alpha=50/number of topics, beta=0.1) while sweeping a grid of topic numbers so that model complexity can be selected by log likelihood.

## When to use

Use when a cisTopic object holds a binary accessibility matrix (e.g. ~97k regulatory regions over 5,335 PBMCs) and multiple LDA models across a topic-number grid must be fitted for downstream model selection.

## When NOT to use

- cisTopic is deprecated — for new analyses prefer pycisTopic, per the repository README
- The input is raw fragment/peak files rather than a binarized accessibility matrix already stored in a cisTopic object — first build the object with createcisTopicObjectFrom10X

## Inputs

- Initialized cisTopic object (RDS) storing a binary accessibility matrix (~97k regulatory regions x 5,335 cells from the 10X 5k PBMC dataset)
- Topic-number grid, e.g. c(2, 5, 10, 15, 20, 25, 30, 35, 40)

## Outputs

- cisTopic object containing 10 fitted LDA topic models across the topic-number grid, saved as an RDS file
- Per-model log-likelihood curves (inspectable via logLikelihoodByIter())

## How to apply

Load the initialized cisTopic object (e.g. cisTopicObject_pbmc.Rds) and call runModels(), which applies LDA with a collapsed Gibbs sampler (Griffiths & Steyvers, 2004, as implemented in the lda package) on the binary accessibility matrix. Sweep topic numbers c(2, 5, 10, 15, 20, 25, 30, 35, 40) with seed=987, nCores=9, burnin=120, iterations=150 and addModels=FALSE, retaining the default Dirichlet hyperparameters alpha=50/number of topics and beta=0.1. Keep all models in memory (default returnType) so that per-model log likelihoods can later be inspected with logLikelihoodByIter() and the best model chosen with selectModel(). Save the object with the 10 fitted models to disk as an RDS file (reference running time for the 5k PBMC dataset: ~80 min).

## Related tools

- **cisTopic** (R/Bioconductor package providing runModels() for LDA topic-model fitting with collapsed Gibbs sampling over the binary accessibility matrix) — http://github.com/aertslab/cisTopic
- **R** (Language in which cisTopic is implemented and the model-fitting workflow is executed)

## Examples

```
cisTopicObject <- runModels(cisTopicObject, topic=c(2,5,10,15,20,25,30,35,40), seed=987, nCores=9, burnin=120, iterations=150, addModels=FALSE)
```

## Evaluation signals

- The saved cisTopic object contains exactly 10 fitted LDA models, one per topic number in the grid c(2, 5, 10, 15, 20, 25, 30, 35, 40)
- Per-model likelihood curves from logLikelihoodByIter() stabilize after the burnin iteration (grey line at 120)
- selectModel() can pick the model with the highest log likelihood P(D|T) at the last iteration from the retained models
- Hyperparameters match the defaults alpha=50/number of topics and beta=0.1, with seed=987 and iterations=150 recorded in the model metadata

## Limitations

- cisTopic is deprecated in favor of pycisTopic
- Model fitting is compute-intensive: reference running time for the 5,335-cell PBMC dataset with ~97k regions is ~80 min
- Fitting multiple topic numbers simultaneously requires retaining all models in memory (default returnType)

## Evidence

- [methods] runModels builds LDA models on binary accessibility matrix: "To do this, `runModels()` builds several models (e.g. with diferent numbers of topics) using Latent Dirichlet Allocation (LDA) on the binary accessibility matrix"
- [methods] logLikelihoodByIter across topic grid: "logLikelihoodByIter(cisTopicObject, select=c(2,5,10,15,20,25,30,35,40))"
- [methods] selectModel selects by highest log likelihood: "`selectModel` will select the model with the highest log likelihood (`P(D|T)`) at the last iteration."
- [readme] cisTopic purpose statement from README: "cisTopic is an R-package to simultaneously identify cell states and cis-regulatory topics from single cell epigenomics data."
