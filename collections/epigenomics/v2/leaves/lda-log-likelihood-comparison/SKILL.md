---
name: lda-log-likelihood-comparison
description: Use when cisTopic has fitted multiple LDA models (e.g. 2–40 topics via
  runModels) on single-cell ATAC-seq data and you must pick one optimal topic number
  for downstream cell-state and cis-regulatory topic analysis.
when_to_use_negative:
- When only a single model with a fixed topic number has been fitted — there is nothing
  to compare and selectModel has no basis for selection
- When log-likelihood curves have not stabilized after burnin — comparisons of final
  likelihoods would be unreliable and models should be refit with more iterations
- When working in the pycisTopic (Python) ecosystem, which replaced the deprecated
  cisTopic R package and uses different model selection tooling
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3673
tools:
- name: cisTopic
  role: R/Bioconductor package providing runModels(), selectModel(), and logLikelihoodByIter()
    for fitting and selecting LDA topic models
  repo: https://github.com/aertslab/cisTopic
- name: R
  role: Language in which the cisTopic model-fitting and selection workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:7a23e531862715ae0f85250689f895ed0f565d780f6ed9f5ad62d831d5d0bb7d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - R
  - cisTopic
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lda-log-likelihood-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Selects the optimal LDA topic model in cisTopic by comparing final-iteration log likelihoods (P(D|T)) across models built with different topic counts, choosing the model with the highest log likelihood (ties broken toward fewer topics).

## When to use

Use when cisTopic has fitted multiple LDA models (e.g. 2–40 topics via runModels) on single-cell ATAC-seq data and you must pick one optimal topic number for downstream cell-state and cis-regulatory topic analysis.

## When NOT to use

- When only a single model with a fixed topic number has been fitted — there is nothing to compare and selectModel has no basis for selection
- When log-likelihood curves have not stabilized after burnin — comparisons of final likelihoods would be unreliable and models should be refit with more iterations
- When working in the pycisTopic (Python) ecosystem, which replaced the deprecated cisTopic R package and uses different model selection tooling

## Inputs

- cisTopic object containing multiple fitted LDA models from runModels() (e.g. topic counts 2, 5, 10, 15, 20, 25, 30, 35, 40; seed 987, burnin 120, iterations 150)
- Binary accessibility matrix from the 10X 5k PBMC dataset (atac_v1_pbmc_5k fragments and peaks) processed via createcisTopicObjectFrom10X()

## Outputs

- Selected optimal cisTopic model (30-topic model for the 5k PBMC vignette)
- Selection table listing per-model final-iteration log likelihoods and the selected model
- Log-likelihood-by-iteration plot (logLikelihoodByIter) showing stabilization after burnin for all topic counts

## How to apply

Fit several LDA models with runModels() on the binary accessibility matrix using a grid of topic counts (in the 5k PBMC vignette: 2, 5, 10, 15, 20, 25, 30, 35, 40, with seed 987, burnin 120, iterations 150). Plot log-likelihood trajectories with logLikelihoodByIter() to confirm each curve stabilizes after the burnin iteration. Then apply selectModel(), which selects the model with the highest log likelihood P(D|T) at the last iteration; ties are broken by choosing the lower number of topics (lower complexity). In the 10X 5k PBMC workflow this procedure selects the 30-topic model. The decision is grounded in the article's stated criterion: model selection is based on the last-iteration log likelihood, not on perplexity or visual inspection alone.

## Related tools

- **cisTopic** (R/Bioconductor package providing runModels(), selectModel(), and logLikelihoodByIter() for fitting and selecting LDA topic models) — https://github.com/aertslab/cisTopic
- **R** (Language in which the cisTopic model-fitting and selection workflow is executed)

## Examples

```
selectModel(cisTopicObject)  # after: logLikelihoodByIter(cisTopicObject, select=c(2,5,10,15,20,25,30,35,40))
```

## Evaluation signals

- The selection table output by selectModel() must list the 30-topic model as selected for the 5k PBMC workflow, matching the vignette's reported choice
- Log-likelihood curves from logLikelihoodByIter() must be stabilized after the burnin iteration (grey line) for every topic count before selection is trusted
- The selected model must be the one with the highest final-iteration P(D|T) among all fitted topic counts
- Ties in final log likelihood must be resolved to the lower number of topics (lower complexity)
- Topic counts fitted must cover the intended grid (2, 5, 10, 15, 20, 25, 30, 35, 40 in the vignette) so the comparison is meaningful

## Limitations

- cisTopic is deprecated; the README instructs users to switch to pycisTopic, so this R-based selection workflow is legacy
- Selection depends on sampler settings (seed 987, burnin 120, iterations 150 in the vignette); insufficient burnin/iterations can leave likelihoods unstable
- Highest log likelihood may not coincide with biologically interpretable topic numbers; the criterion is purely likelihood-based

## Evidence
