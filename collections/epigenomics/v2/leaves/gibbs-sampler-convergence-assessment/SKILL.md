---
name: gibbs-sampler-convergence-assessment
description: Use when runModels has built several LDA models (e.g. 2–40 topics) on
  single-cell ATAC-seq data and you must verify the log-likelihood stabilized after
  burnin and pick the model.
when_to_use_negative:
- cisTopic is deprecated in favor of pycisTopic; for new Python-based workflows use
  pycisTopic's model selection instead
- Input is a single pre-trained model with only one topic count — there is nothing
  to compare and selectModel's cross-model criterion does not apply
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: cisTopic
  role: R/Bioconductor package providing runModels(), logLikelihoodByIter(), and selectModel()
    for LDA model fitting, convergence plotting, and selection
  repo: http://github.com/aertslab/cistopic
- name: pycisTopic
  role: Deprecated cisTopic's suggested successor for Python-based topic modeling
    workflows
  repo: https://github.com/aertslab/pycisTopic
- name: R
  role: Language in which the cisTopic model selection workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:33937efa3b12f28123df0a9ddc64235d01cc56343f1f39de086d59845b86a959
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  - pycisTopic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gibbs-sampler-convergence-assessment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Assess convergence of cisTopic's LDA (WarpLDA) Gibbs-sampler runs and select the optimal topic model by comparing final-iteration log likelihood P(D|T) across models fitted with different numbers of topics. For the 5k PBMC 10X dataset, this criterion selects the 30-topic model.

## When to use

Use when runModels has built several LDA models (e.g. 2–40 topics) on single-cell ATAC-seq data and you must verify the log-likelihood stabilized after burnin and pick the best model.

## When NOT to use

- cisTopic is deprecated in favor of pycisTopic; for new Python-based workflows use pycisTopic's model selection instead
- Input is a single pre-trained model with only one topic count — there is nothing to compare and selectModel's cross-model criterion does not apply

## Inputs

- cisTopic object (RDS, e.g. cisTopicObject_pbmc.Rds) containing multiple fitted LDA models from runModels()
- Model specifications with differing topic counts (2, 5, 10, 15, 20, 25, 30, 35, 40), seed 987, burnin 120, iterations 150

## Outputs

- Log-likelihood-per-iteration trajectories for each topic count (logLikelihoodByIter plot)
- Per-model final-iteration log likelihood P(D|T) values
- Selection table naming the single optimal model (30 topics for the 5k PBMC vignette)
- Selected cisTopic model used for downstream cell state / topic analysis

## How to apply

After runModels() fits LDA models at topic counts 2, 5, 10, 15, 20, 25, 30, 35, 40 (e.g. seed 987, burnin 120, iterations 150) on the cisTopicObject, plot log-likelihood trajectories with logLikelihoodByIter() and confirm each curve is flat after the burnin iteration (grey line), indicating the Gibbs sampler has converged. Then extract the last-iteration log likelihood P(D|T) per topic count and apply selectModel(), which picks the model with the highest final log likelihood, breaking ties toward fewer topics (lower complexity). For the 5k PBMC dataset (5,335 cells, 97k potential regulatory regions), this procedure selects the 30-topic model. The selected topic count is then used downstream for cell state and cis-regulatory topic analysis.

## Related tools

- **cisTopic** (R/Bioconductor package providing runModels(), logLikelihoodByIter(), and selectModel() for LDA model fitting, convergence plotting, and selection) — http://github.com/aertslab/cistopic
- **pycisTopic** (Deprecated cisTopic's suggested successor for Python-based topic modeling workflows) — https://github.com/aertslab/pycisTopic
- **R** (Language in which the cisTopic model selection workflow is executed)

## Examples

```
logLikelihoodByIter(cisTopicObject, select=c(2,5,10,15,20,25,30,35,40))
```

## Evaluation signals

- Log-likelihood curves for every topic count must be visibly stabilized after the burnin iteration (grey line) before selection is trusted
- The selection table output by selectModel() must list the 30-topic model as selected for the 5k PBMC dataset
- The chosen model must have the highest final-iteration P(D|T) among all fitted topic counts
- All models compared must share the same sampling settings (e.g. seed 987, burnin 120, iterations 150) so likelihoods are comparable

## Limitations

- Selection relies solely on highest final-iteration log likelihood; ties are broken toward fewer topics rather than by biological interpretability
- Convergence is judged visually/empirically from trajectory plots; if burnin or iterations are too short, curves may not be stabilized and selection is unreliable
- cisTopic itself is deprecated; the maintainers direct users to pycisTopic for continued use

## Evidence

- [methods] To do this, `runModels()` builds several models (e.g. with diferent numbers of topics) using Latent Dirichlet Allocation (LDA) on the binary accessibility matrix: "To do this, `runModels()` builds several models (e.g. with diferent numbers of topics) using Latent Dirichlet Allocation (LDA) on the binary accessibility matrix"
- [methods] `selectModel` will select the model with the highest log likelihood (`P(D|T)`) at the last iteration.: "`selectModel` will select the model with the highest log likelihood (`P(D|T)`) at the last iteration."
- [methods] logLikelihoodByIter(cisTopicObject, select=c(2,5,10,15,20,25,30,35,40)): "logLikelihoodByIter(cisTopicObject, select=c(2,5,10,15,20,25,30,35,40))"
- [readme] cisTopic is an R-package to simultaneously identify cell states and cis-regulatory topics from single cell epigenomics data.: "cisTopic is an R-package to simultaneously identify cell states and cis-regulatory topics from single cell epigenomics data."
