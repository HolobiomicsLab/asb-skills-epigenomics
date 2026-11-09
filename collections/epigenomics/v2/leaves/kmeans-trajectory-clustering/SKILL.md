---
name: kmeans-trajectory-clustering
description: Use when a normalized feature-by-donor abundance matrix (peaks or genes)
  has samples ordered by age and you need to detect and classify features showing
  genuine monotonic or non-monotonic temporal trends over adult lifespan.
when_to_use_negative:
- Input is a single-cell or sorted-cell dataset — the article's bulk PBMC annotation
  approach is acknowledged as prone to reference biases and the authors note single-cell
  resolution sex-difference descriptions were not established
- Samples are not age-ordered or lack donor age metadata, so a chronological series
  per feature cannot be constructed
- You want static differential abundance between age groups (e.g., older vs younger
  contrasts) rather than per-feature longitudinal trajectory detection and clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics: []
tools:
- name: R Shiny
  role: Interactive, searchable delivery of the resulting immune-aging trajectories
    and analyses to users (https://immune-aging.jax.org/)
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:f7170bec8e49e979293e18336a1ae9eb359d5772f3e348e3a0c4078e83b5b6a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kmeans-trajectory-clustering

## Summary

Identifies longitudinal (chronological) genomic trends from PBMC ATAC-seq and RNA-seq feature-by-donor matrices by fitting auto.arima models across an age-ordered series, filtering for features with real temporal trends, and grouping their z-scored fitted trajectories with k-means (k=3) into biologically interpretable aging patterns.

## When to use

Use when a normalized feature-by-donor abundance matrix (peaks or genes) has samples ordered by age and you need to detect and classify features showing genuine monotonic or non-monotonic temporal trends over adult lifespan.

## When NOT to use

- Input is a single-cell or sorted-cell dataset — the article's bulk PBMC annotation approach is acknowledged as prone to reference biases and the authors note single-cell resolution sex-difference descriptions were not established
- Samples are not age-ordered or lack donor age metadata, so a chronological series per feature cannot be constructed
- You want static differential abundance between age groups (e.g., older vs younger contrasts) rather than per-feature longitudinal trajectory detection and clustering

## Inputs

- Normalized feature-by-donor abundance matrix (ATAC-seq peaks or RNA-seq genes across PBMC donors from 172 healthy adults aged 22-93)
- Donor metadata with ages used to order sample columns chronologically

## Outputs

- trajectory_clustering_results.csv containing k-means cluster assignments per retained feature
- Synthetic-benchmark recall and precision metrics for ARIMA-based trend recovery
- k=3 aging trajectory clusters: declining naive, increasing monocyte/cytotoxic, and stable/other

## How to apply

Load the normalized feature-by-donor abundance matrix from the repository data files and order the donor columns by age. For each feature, fit an auto.arima model (forecast package in R) across the age-ordered series and select the model minimizing the AIC. Filter the fitted features, retaining only those whose selected ARIMA order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0), indicating a longitudinal trend rather than random fluctuation. Z-score the auto.arima fitted values of each retained feature across the age axis, then cluster the z-scored fitted trajectories with k-means using k=3 — the biologically informed cluster count corresponding to declining naive T cell trajectories, increasing monocyte/cytotoxic trajectories, and stable/other patterns. Validate by generating synthetic age-ordered series with implanted monotonic and non-monotonic trends, running the same selector on them, and computing recall and precision of trended-feature recovery.

## Related tools

- **R Shiny** (Interactive, searchable delivery of the resulting immune-aging trajectories and analyses to users (https://immune-aging.jax.org/)) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
fit <- forecast::auto.arima(feature_series); keep <- sum(unlist(purrr::map(fit$arma, abs)))>0 & fit$arma[6]>0; km <- kmeans(scale(fitted(fit)[keep,]), centers=3)
```

## Evaluation signals

- Only features whose selected auto.arima order satisfies D>0 AND (AR+MA)>0 are passed to clustering; check this filter invariant before k-means
- k-means must use k=3; the three clusters should be interpretable as declining naive T cell, increasing monocyte/cytotoxic, and stable/other trajectories
- trajectory_clustering_results.csv is produced and contains cluster assignments for every retained feature
- Synthetic benchmark: series with implanted monotonic and non-monotonic trends should show high recall; random fluctuation-only series should yield low precision loss (high specificity)
- Fitted values are z-scored across the age axis before clustering — verify clustering is run on z-scored auto.arima fitted values, not raw abundances

## Limitations

- The differencing/AR-MA order filter is a heuristic for trend detection; its recall and precision must be estimated via synthetic series with implanted trends
- k=3 is a biologically informed but externally imposed cluster count tied to the paper's major aging patterns; other datasets may need a different k
- The bulk PBMC feature trajectories confound cell-composition shifts with cell-intrinsic changes; the article notes not all sex-biased differences in older adults are attributable to cell composition

## Evidence

- [methods] ARIMA fit and AIC selection: [evidence span withheld — anchor status: not_found]
- [methods] Trend filter step: [evidence span withheld — anchor status: not_found]
- [methods] k=3 k-means clustering: [evidence span withheld — anchor status: not_found]
- [abstract] Biological interpretation of clusters: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [intro] Trajectory analysis rationale: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [readme] Repository scope: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
