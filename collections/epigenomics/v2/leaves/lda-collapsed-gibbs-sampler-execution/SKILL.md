---
name: lda-collapsed-gibbs-sampler-execution
description: Use when a cisTopic object holding a binarized accessibility matrix (e.g.
  5,335 PBMC cells × ~97k regions) is ready and a sweep of LDA topic models across
  a topic-number grid is required for subsequent model selection.
when_to_use_negative:
- The cisTopic object has not yet been initialized from the 10X fragments/peaks files
  (run createcisTopicObjectFrom10X first) so no binary accessibility matrix exists
- Only a single topic number is needed with no plan to compare models by log likelihood
  — running the full 10-model grid wastes ~80 min of compute
- 'Working in Python or with newer pipelines: the R cisTopic package is deprecated
  in favor of pycisTopic'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: cisTopic
  role: R/Bioconductor package whose runModels() function fits LDA models with a collapsed
    Gibbs sampler on the binary accessibility matrix; installed via devtools::install_github("aertslab/cisTopic")
  repo: https://github.com/aertslab/cistopic
- name: R
  role: Language/runtime in which cisTopic is executed
- name: pycisTopic
  role: Python successor to cisTopic recommended by the README as the replacement
    for the deprecated R package
  repo: https://github.com/aertslab/pycisTopic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:fb0f549bcc08afca8d2a4bf387448008504eda0af5ef9c5d8bcc4b5f1b4b5a09
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cistopic
  tools:
  - R
  - cisTopic
  - pycisTopic
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lda-collapsed-gibbs-sampler-execution

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Fit a grid of Latent Dirichlet Allocation topic models on a binary single-cell accessibility matrix using cisTopic's runModels(), which applies LDA with a collapsed Gibbs sampler (Griffiths & Steyvers, 2004, via the lda package) over a range of topic numbers so model selection by log likelihood is possible downstream.

## When to use

Use when a cisTopic object holding a binarized accessibility matrix (e.g. 5,335 PBMC cells × ~97k regions) is ready and a sweep of LDA topic models across a topic-number grid is required for subsequent model selection.

## When NOT to use

- The cisTopic object has not yet been initialized from the 10X fragments/peaks files (run createcisTopicObjectFrom10X first) so no binary accessibility matrix exists
- Only a single topic number is needed with no plan to compare models by log likelihood — running the full 10-model grid wastes ~80 min of compute
- Working in Python or with newer pipelines: the R cisTopic package is deprecated in favor of pycisTopic

## Inputs

- Initialized cisTopic object (RDS) storing a binary accessibility matrix, e.g. 5,335-cell PBMC dataset with ~97k regulatory regions
- Topic-number grid, here c(2, 5, 10, 15, 20, 25, 30, 35, 40)

## Outputs

- cisTopic object containing 10 fitted LDA models over the requested topic-number grid
- Per-model log likelihood curves inspectable with logLikelihoodByIter()
- Saved RDS file with all fitted models for downstream model selection

## How to apply

Load the initialized cisTopic object (e.g. cisTopicObject_pbmc.Rds from the 10X 5k PBMC workflow) and call runModels(), which builds models with different numbers of topics using LDA on the binary accessibility matrix with a collapsed Gibbs sampler. Fit the topic-number grid c(2, 5, 10, 15, 20, 25, 30, 35, 40) with seed=987, nCores=9, burnin=120, iterations=150 and addModels=FALSE, keeping the default Dirichlet hyperparameters alpha=50/number of topics and beta=0.1. Retain all fitted models in memory (default returnType) so that logLikelihoodByIter() and selectModel() can compare them later. Finally save the object with the 10 fitted models as an RDS file; reference running time on the 5k PBMC dataset is ~80 min.

## Related tools

- **cisTopic** (R/Bioconductor package whose runModels() function fits LDA models with a collapsed Gibbs sampler on the binary accessibility matrix; installed via devtools::install_github("aertslab/cisTopic")) — https://github.com/aertslab/cistopic
- **R** (Language/runtime in which cisTopic is executed)
- **pycisTopic** (Python successor to cisTopic recommended by the README as the replacement for the deprecated R package) — https://github.com/aertslab/pycisTopic

## Examples

```
cisTopicObject <- readRDS('cisTopicObject_pbmc.Rds'); cisTopicObject <- runModels(cisTopicObject, topic=c(2,5,10,15,20,25,30,35,40), seed=987, nCores=9, burnin=120, iterations=150, addModels=FALSE)
```

## Evaluation signals

- The saved object contains exactly 10 fitted LDA models corresponding to topic numbers 2, 5, 10, 15, 20, 25, 30, 35 and 40
- logLikelihoodByIter() curves for each model stabilize after the burnin iteration (grey line at 120)
- selectModel() can identify the model with the highest log likelihood P(D|T) at the last iteration, confirming all models were retained
- Run completes without adding to prior models (addModels=FALSE) and the resulting RDS file loads back with readRDS()

## Limitations

- Reference running time is ~80 min for the 5,335-cell / ~97k-region PBMC dataset, scaling with cells × regions and the topic grid size
- The R cisTopic package is deprecated; new projects should use pycisTopic instead
- Only likelihood-based selection is supported here; downstream interpretation depends on choosing an adequate topic number from the grid (capped at 40 in this workflow)

## Evidence
