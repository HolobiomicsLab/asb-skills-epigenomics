---
name: multi-topic-grid-model-training
description: Use when a cisTopic object (initialized binary accessibility matrix)
  is available and multiple LDA models over a grid of topic numbers must be trained
  to later choose the optimal number of cis-regulatory topics.
when_to_use_negative:
- Input is not a cisTopic object with a binarized accessibility matrix (e.g. raw count
  matrix not yet initialized via createcisTopicObject*)
- The optimal model has already been selected and downstream steps (getRegionsScores,
  binarizecisTopics, runtSNE/runUmap) are the goal
- Working in Python with the deprecated R cisTopic replaced by pycisTopic, where runModels()
  is not the applicable API
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0769
tools:
- name: cisTopic
  role: R/Bioconductor package whose runModels() function fits the LDA topic models
    over the topic-number grid on the binary accessibility matrix; runModels, selectModel
    and logLikelihoodByIter are the relevant functions
  repo: http://github.com/aertslab/cistopic
- name: R
  role: Runtime environment in which the cisTopic package is executed
- name: lda
  role: Package providing the collapsed Gibbs sampler implementation of LDA used by
    runModels()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:cf69c84edc9eb5ff33d680db190f6ae6d2b7f680135e75dab5acab19dacb67bd
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  - lda
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Multi-topic-grid model training

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Fits a series of Latent Dirichlet Allocation (LDA) topic models over a range of topic numbers on a cisTopic object built from single-cell epigenomics data, so that the best topic number can later be selected by log likelihood. In this workflow, runModels() is applied to the binary accessibility matrix of the 5,335-cell PBMC dataset (~97k regulatory regions) across a 2–40 topic grid.

## When to use

Use when a cisTopic object (initialized binary accessibility matrix) is available and multiple LDA models over a grid of topic numbers must be trained to later choose the optimal number of cis-regulatory topics.

## When NOT to use

- Input is not a cisTopic object with a binarized accessibility matrix (e.g. raw count matrix not yet initialized via createcisTopicObject*)
- The optimal model has already been selected and downstream steps (getRegionsScores, binarizecisTopics, runtSNE/runUmap) are the goal
- Working in Python with the deprecated R cisTopic replaced by pycisTopic, where runModels() is not the applicable API

## Inputs

- Initialized cisTopic object (RDS, e.g. cisTopicObject_pbmc.Rds) storing the binary accessibility matrix of ~97k regulatory regions for 5,335 PBMCs
- Grid of topic numbers, e.g. c(2, 5, 10, 15, 20, 25, 30, 35, 40)
- Sampling parameters: seed=987, nCores=9, burnin=120, iterations=150, addModels=FALSE

## Outputs

- cisTopic object containing 10 fitted LDA topic models retained in memory (default returnType)
- Saved RDS file of the cisTopic object with all fitted models
- Per-model log likelihood curves inspectable via logLikelihoodByIter()

## How to apply

Load the initialized cisTopic object (e.g. cisTopicObject_pbmc.Rds from the 10X 5k PBMCs dataset) and call runModels() to fit LDA with a collapsed Gibbs sampler (Griffiths & Steyvers, 2004, as implemented in the lda package) on the binary accessibility matrix. Fit models with topic numbers c(2, 5, 10, 15, 20, 25, 30, 35, 40) using seed=987, nCores=9, burnin=120, iterations=150 and addModels=FALSE, keeping the default Dirichlet hyperparameters alpha=50/number of topics and beta=0.1. Keep the default returnType so all models remain in memory simultaneously, enabling later model selection via log likelihood (logLikelihoodByIter()) or selectModel (highest log likelihood P(D|T) at the last iteration). Finally save the object with all 10 fitted models to disk as an RDS file; reference running time for the 5,335-cell PBMC dataset is ~80 minutes.

## Related tools

- **cisTopic** (R/Bioconductor package whose runModels() function fits the LDA topic models over the topic-number grid on the binary accessibility matrix; runModels, selectModel and logLikelihoodByIter are the relevant functions) — http://github.com/aertslab/cistopic
- **R** (Runtime environment in which the cisTopic package is executed)
- **lda** (Package providing the collapsed Gibbs sampler implementation of LDA used by runModels())

## Examples

```
cisTopicObject <- readRDS('cisTopicObject_pbmc.Rds'); cisTopicObject <- runModels(cisTopicObject, topic=c(2, 5, 10, 15, 20, 25, 30, 35, 40), seed=987, nCores=9, burnin=120, iterations=150, addModels=FALSE)
```

## Evaluation signals

- The saved object contains exactly 10 fitted LDA models corresponding to the grid c(2, 5, 10, 15, 20, 25, 30, 35, 40)
- logLikelihoodByIter() yields per-model likelihood curves that stabilize after the burnin iteration (grey line at 120)
- Dirichlet hyperparameters follow the defaults alpha=50/number of topics and beta=0.1
- All models are retained simultaneously in memory (default returnType) so selectModel can pick the highest log likelihood P(D|T) at the last iteration
- Output RDS file is readable with readRDS and runs complete within the reference ~80 min for the 5,335-cell PBMC dataset

## Limitations

- cisTopic (R) is deprecated in favor of pycisTopic
- Runtime scales with dataset size: ~80 min for 5,335 cells over ~97k regions with 9 cores; grid of 10 topic values multiplies cost
- Models are kept in memory simultaneously (default returnType), which can be demanding for large datasets

## Evidence
