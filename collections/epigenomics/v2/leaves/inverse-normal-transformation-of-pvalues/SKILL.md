---
name: inverse-normal-transformation-of-pvalues
description: 'Use when combining non-independent p-values (e.g. correlated methylation
  promoter/track p-values within a genomic interval) and you need a valid Stouffer-style
  combination: convert p to Z with qnorm(1 - p/2), then decorrelate using the Cholesky
  factor of Sigma.'
when_to_use_negative:
- P-values are already independent — plain Stouffer/Fisher combination suffices and
  Cholesky whitening is unnecessary
- Sigma is not symmetric positive-definite (e.g. too few samples relative to correlated
  features) — chol() will fail and the whitening is invalid
- P-values are one-sided or have unknown directionality — the Phi^{-1}(1 - p/2) two-sided
  mapping would miscode the effect direction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3940
tools:
- name: SMITE
  role: Implements the correlation-adjusted Stouffer combination (stoufferTest, annotateModification
    with mod_corr=TRUE) that the inverse-normal transformation and Cholesky whitening
    mirror
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Execution environment providing qnorm for Phi^{-1}(1 - p/2), chol() for the
    factorization Sigma = C^T C, and the SMITE package data(methylation)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:becd412385b22c94e21bc64ef4cbc101b4028d3b28207d6112a1bc78a213837f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# inverse-normal-transformation-of-pvalues

## Summary

Transforms two-sided p-values into signed Z-scores via the inverse normal CDF (Phi^{-1}(1 - p/2), qnorm in R) and, when the p-values are correlated as in SMITE's methylation contexts, whitens them with the inverse Cholesky factor of the correlation matrix (Sigma = C^T C) so the resulting Z* scores are approximately independent and can be combined as if from independent experiments.

## When to use

Use when combining non-independent p-values (e.g. correlated methylation promoter/track p-values within a genomic interval) and you need a valid Stouffer-style combination: convert p to Z with qnorm(1 - p/2), then decorrelate using the Cholesky factor of Sigma.

## When NOT to use

- P-values are already independent — plain Stouffer/Fisher combination suffices and Cholesky whitening is unnecessary
- Sigma is not symmetric positive-definite (e.g. too few samples relative to correlated features) — chol() will fail and the whitening is invalid
- P-values are one-sided or have unknown directionality — the Phi^{-1}(1 - p/2) two-sided mapping would miscode the effect direction

## Inputs

- Matrix of two-sided p-values with gene identifiers and per-context columns (e.g. methylation promoter and track p-values)
- Correlation structure Sigma among the non-independent p-values (as constructed by annotateModification with mod_corr=TRUE)

## Outputs

- Whitened Z-score table with gene identifiers and per-context Z* columns (e.g. whitened_z_scores.csv)
- Empirical off-diagonal correlation check showing Z* columns are approximately independent

## How to apply

Load the p-value matrix (e.g. SMITE's packaged methylation data via data(methylation)), first removing NAs and replacing zeros with the minimum non-zero p-value. Build the correlation structure among the non-independent p-values (mod_corr=TRUE behavior of annotateModification) and confirm Sigma is symmetric positive-definite before factorizing Sigma = C^T C with R's chol(). Transform each two-sided p-value with Phi^{-1}(1 - p/2) (qnorm in R) and whiten with the inverse Cholesky factor, Z* = C^{-1} Phi^{-1}(1 - p/2), mirroring the Stouffer-based correlated combination implemented in stoufferTest. Validate by checking that off-diagonal correlations among the whitened Z* columns are near zero, replicating the Monte Carlo random-sampling (MCM) check SMITE uses to derive FDR-like p-values.

## Related tools

- **SMITE** (Implements the correlation-adjusted Stouffer combination (stoufferTest, annotateModification with mod_corr=TRUE) that the inverse-normal transformation and Cholesky whitening mirror) — https://github.com/GreallyLab/SMITE
- **R** (Execution environment providing qnorm for Phi^{-1}(1 - p/2), chol() for the factorization Sigma = C^T C, and the SMITE package data(methylation))

## Examples

```
Z <- qnorm(1 - methylation[,5]/2); C <- chol(Sigma); Zstar <- solve(C, Z)
```

## Evaluation signals

- Script runs without error and whitened_z_scores.csv exists with the expected gene and Z* columns
- Empirical off-diagonal correlations of the whitened Z* columns fall near zero (within Monte Carlo sampling variability)
- Whitened scores match Z* = C^{-1} Phi^{-1}(1 - p/2) applied with the chol() factor of Sigma, mirroring stoufferTest behavior
- Sigma passed the symmetric positive-definite check before Cholesky factorization

## Limitations

- Correlated values are only approximately independent after whitening; SMITE additionally uses Monte Carlo random sampling (MCM) to derive FDR-like p-values
- Requires an estimable symmetric positive-definite correlation structure; poorly estimated Sigma degrades the decorrelation

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [methods] test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer": "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer""
- [methods] Replace zeros with minimum non-zero p-value: "## Replace zeros with minimum non-zero p-value ##"
