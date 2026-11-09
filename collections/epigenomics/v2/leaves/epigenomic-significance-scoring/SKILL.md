---
name: epigenomic-significance-scoring
description: Use when a genomic interval has multiple non-independent p-values (e.g.
  correlated methylation or H3K4me1 probes within one gene) that must be combined
  into a single valid gene-level significance score.
when_to_use_negative:
- The p-values to combine are already independent — plain Stouffer/Fisher combination
  suffices and Cholesky whitening is unnecessary
- The correlation matrix Sigma is not symmetric positive-definite (e.g. fewer observations
  than probes), making the Cholesky factorization undefined
- The goal is differential module detection from pre-scored genes with no raw p-values
  to combine — use module-finding tools instead
tools:
- name: SMITE
  role: Implements correlation-aware Stouffer p-value combination (annotateModification
    with mod_corr=TRUE), normalization (normalizePval), and weighted gene scoring
    (scorePval)
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Execution environment; uses chol() for Cholesky factorization of Sigma and
    qnorm for Phi^{-1}(1 - p/2) transformation
- name: igraph
  role: Used after scoring to run the spinglass algorithm and determine network modules
    from the scored genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:f7590ca7be8b53a9f4838cb57986e7c6ae4deaf6e8500b0cc001d23792aa0d11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# epigenomic-significance-scoring

## Summary

Assign per-gene significance scores by combining correlated epigenomic (e.g. methylation, histone-mark) and transcriptomic p-values, correcting non-independence via a Cholesky-whitened, Stouffer-based combination as implemented in the SMITE R/Bioconductor package.

## When to use

Use when a genomic interval has multiple non-independent p-values (e.g. correlated methylation or H3K4me1 probes within one gene) that must be combined into a single valid gene-level significance score.

## When NOT to use

- The p-values to combine are already independent — plain Stouffer/Fisher combination suffices and Cholesky whitening is unnecessary
- The correlation matrix Sigma is not symmetric positive-definite (e.g. fewer observations than probes), making the Cholesky factorization undefined
- The goal is differential module detection from pre-scored genes with no raw p-values to combine — use module-finding tools instead

## Inputs

- methylation p-value matrix (CpG-level two-sided p-values per gene/interval, e.g. SMITE's packaged methylation data)
- curated expression p-value data (e.g. data(curated_expressiondata))
- histone modification p-values (e.g. data(histone_h3k4me1))
- correlation structure among non-independent p-values within each genomic interval (Sigma, symmetric positive-definite)

## Outputs

- per-gene combined significance scores (scorePval-weighted p-values) attached to the PvalueAnnotation object
- whitened Z-score table with gene identifiers and per-context Z* columns (whitened_z_scores.csv)
- gene-level significance values for downstream network module scoring

## How to apply

Load packaged methylation p-values (data(methylation) in SMITE), remove NAs, and replace zeros with the minimum non-zero p-value before any transformation. Annotate modifications with annotateModification using weight_by_method="Stouffer" and mod_corr=TRUE, building a symmetric positive-definite correlation matrix Sigma among the non-independent p-values. Transform each two-sided p-value to a Z-score via qnorm(1 - p/2), then whiten with the inverse Cholesky factor of Sigma (Z* = C^{-1} Phi^{-1}(1 - p/2)) so correlated scores become approximately independent and can be combined as independent experiments. Normalize p-values with normalizePval(method="rescale", ref="expression"), then produce final gene scores with scorePval applying data-type weights (e.g. methylation_promoter=.3), and validate by confirming off-diagonal correlations of whitened Z* columns fall near zero within Monte Carlo (MCM) sampling variability.

## Related tools

- **SMITE** (Implements correlation-aware Stouffer p-value combination (annotateModification with mod_corr=TRUE), normalization (normalizePval), and weighted gene scoring (scorePval)) — https://github.com/GreallyLab/SMITE
- **R** (Execution environment; uses chol() for Cholesky factorization of Sigma and qnorm for Phi^{-1}(1 - p/2) transformation)
- **igraph** (Used after scoring to run the spinglass algorithm and determine network modules from the scored genes)

## Examples

```
test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer", mod_corr=TRUE)
```

## Evaluation signals

- The correlation matrix Sigma built from the p-values is confirmed symmetric positive-definite so chol() succeeds
- Empirical off-diagonal correlations of the whitened Z* scores fall near zero, within Monte Carlo random-sampling (MCM) variability
- The pipeline runs without error and whitened_z_scores.csv exists with expected gene identifiers and per-context Z* columns
- Data hygiene filters applied: no NAs remain in the p-value column and zero p-values are replaced by the minimum non-zero p-value
- Final gene scores reflect the specified weights (e.g. methylation_promoter=.3) in scorePval

## Limitations

- Correlation adjustment requires enough observations to estimate a symmetric positive-definite Sigma within the interval; degenerate correlation structures break the Cholesky factorization
- Whitened scores are only approximately independent, so combined p-values are validated against MCM-derived FDR-like expectations rather than exact distributions
- P-value preprocessing (NA removal, zero-replacement with the minimum non-zero value) alters raw p-values before combination and must be applied consistently

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [methods] test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer",: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [discussion] Modules implicate potentially important network members for which there may be no statistically significant evidence.: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
