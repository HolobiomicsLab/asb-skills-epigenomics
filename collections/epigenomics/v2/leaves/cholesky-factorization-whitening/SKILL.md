---
name: cholesky-factorization-whitening
description: Use when combining two-sided p-values (e.g. methylation promoter/other-context
  p-values in a genomic interval) that are correlated, so Stouffer-style combination
  would otherwise be invalid.
when_to_use_negative:
- The p-values being combined are already independent — plain Stouffer combination
  suffices and whitening adds unnecessary error
- The correlation matrix Sigma is not symmetric positive-definite (e.g. pairwise-correlation
  estimates with missing values), so chol() will fail and the whitening transform
  is undefined
- The p-values are one-sided or you intend to combine untransformed p-values rather
  than normal-quantile-transformed Z scores
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: SMITE
  role: R/Bioconductor package whose stoufferTest and annotateModification (mod_corr=TRUE,
    weight_by_method='Stouffer') implement the correlation-adjusted Stouffer combination
    this whitening step underpins
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Language in which chol() and qnorm() are used to compute the Cholesky factor
    and the normal-quantile transform of p-values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:6c3a790d6643bea339261e145e72ced2ed6f6bf10fcbd87540e44cc743117c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cholesky-factorization-whitening

## Summary

Whitens correlated, two-sided p-values so they can be combined as independent experiments: transform p to Z via Phi^-1(1-p/2), then apply the inverse Cholesky factor of the correlation matrix Sigma = C^T C, as SMITE does in its Stouffer-based correlated p-value combination.

## When to use

Use when combining two-sided p-values (e.g. methylation promoter/other-context p-values in a genomic interval) that are correlated, so Stouffer-style combination would otherwise be invalid.

## When NOT to use

- The p-values being combined are already independent — plain Stouffer combination suffices and whitening adds unnecessary error
- The correlation matrix Sigma is not symmetric positive-definite (e.g. pairwise-correlation estimates with missing values), so chol() will fail and the whitening transform is undefined
- The p-values are one-sided or you intend to combine untransformed p-values rather than normal-quantile-transformed Z scores

## Inputs

- Matrix of two-sided p-values (e.g. packaged methylation p-value data from SMITE, loaded via data(methylation)) with gene identifiers and per-context columns
- Symmetric positive-definite correlation matrix Sigma among the non-independent p-values

## Outputs

- Table of whitened Z-scores (Z* = C^{-1} Phi^-1(1 - p/2)) with gene identifiers and per-context Z* columns, saved to whitened_z_scores.csv
- Empirical off-diagonal correlation check showing near-zero correlations among whitened Z* columns

## How to apply

Build the correlation matrix Sigma among the non-independent p-values (the mod_corr=TRUE behavior of SMITE's annotateModification) and confirm Sigma is symmetric positive-definite. Compute the Cholesky factorization Sigma = C^T C with R's chol(). Transform each two-sided p-value via Z = Phi^-1(1 - p/2) using qnorm, then whiten with the inverse Cholesky factor, Z* = C^{-1} Z, so the resulting scores are approximately independent and can be combined as independent experiments, mirroring SMITE's stoufferTest. Validate by checking off-diagonal correlations among the Z* columns are near zero, replicating SMITE's Monte Carlo random-sampling (MCM) check used to derive FDR-like p-values, then save the whitened Z-score table with gene identifiers and per-context Z* columns.

## Related tools

- **SMITE** (R/Bioconductor package whose stoufferTest and annotateModification (mod_corr=TRUE, weight_by_method='Stouffer') implement the correlation-adjusted Stouffer combination this whitening step underpins) — https://github.com/GreallyLab/SMITE
- **R** (Language in which chol() and qnorm() are used to compute the Cholesky factor and the normal-quantile transform of p-values)

## Examples

```
C <- chol(Sigma); Z <- qnorm(1 - p/2); Z_star <- solve(C, Z)
```

## Evaluation signals

- Sigma must be symmetric positive-definite so chol(Sigma) runs without error
- whitened_z_scores.csv exists with the expected gene identifier and per-context Z* columns
- Empirical off-diagonal correlations of the whitened Z* columns fall near zero (within Monte Carlo sampling variability), consistent with SMITE's MCM-derived FDR-like p-value check
- The script completes end-to-end without error on the packaged data(methylation) p-value matrix

## Limitations

- Whitening only yields approximately independent scores; residual dependence is absorbed into the Monte Carlo random-sampling (MCM) variability used for FDR-like p-values
- Requires a valid symmetric positive-definite correlation estimate among the non-independent p-values; genomic experiments are often underpowered, weakening correlation estimates

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [methods] annotateModification with Stouffer weighting: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [intro] Need for flexible integration into a single score: "there is a need for a flexible method integrating genomic assay data into a single score that can be used to identify functionally important pathways for further study."
