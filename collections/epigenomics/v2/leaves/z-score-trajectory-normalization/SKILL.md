---
name: z-score-trajectory-normalization
description: Use when a feature-by-donor abundance matrix has an ordered age/time
  axis and you need to normalize per-feature fitted temporal trends to comparable
  scale before clustering, e.g. finding shared aging trajectories of peaks/genes across
  adult lifespan.
when_to_use_negative:
- The feature series have no meaningful chronological/time ordering — z-scoring and
  k-means on unordered samples would not capture longitudinal trends.
- The input is raw or already-interpreted cluster labels rather than a normalized
  abundance matrix requiring per-feature trend fitting.
- Short series where ARIMA fitting is unreliable; the paper's design assumes one value
  per donor across a 22-93 year adult lifespan span.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R Shiny
  role: Interactive web application used to share and explore the immune-aging results,
    including the longitudinal genomic trends derived from this trajectory normalization/clustering
    pipeline
  repo: https://immune-aging.jax.org/
- name: forecast (auto.arima)
  role: R package used to fit the best AIC-minimizing ARIMA model per feature and
    supply the fitted values that are z-scored prior to k-means clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:c166244e3c7cc7a5ad133198e6636f50cbf477ef27d96972984e8eaff6e12789
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - forecast (auto.arima)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# z-score-trajectory-normalization

## Summary

A trajectory-normalization and clustering skill from PBMC ATAC-seq/RNA-seq aging analysis: features passing an auto.arima longitudinal-trend filter (differencing D>0 plus at least one AR or MA term) have their fitted age-series z-scored and then grouped with k-means (k=3) to recover the major aging trajectories.

## When to use

Use when a feature-by-donor abundance matrix has an ordered age/time axis and you need to normalize per-feature fitted temporal trends to comparable scale before clustering, e.g. finding shared aging trajectories of peaks/genes across adult lifespan.

## When NOT to use

- The feature series have no meaningful chronological/time ordering — z-scoring and k-means on unordered samples would not capture longitudinal trends.
- The input is raw or already-interpreted cluster labels rather than a normalized abundance matrix requiring per-feature trend fitting.
- Short series where ARIMA fitting is unreliable; the paper's design assumes one value per donor across a 22-93 year adult lifespan span.

## Inputs

- Normalized feature-by-donor abundance matrix (ATAC-seq peaks or RNA-seq genes across PBMC donors aged 22-93)
- Donor age metadata used to order the series columns chronologically
- Fitted values from per-feature auto.arima models (forecast package, R)

## Outputs

- Z-scored per-feature trajectory matrix of auto.arima fitted values across the age axis
- k-means (k=3) cluster assignments per feature (declining naive, increasing monocyte/cytotoxic, stable/other)
- Synthetic-benchmark recall and precision metrics of trend recovery
- trajectory_clustering_results.csv combining cluster assignments and recall/precision metrics

## How to apply

Load the normalized feature-by-donor matrix and order donor columns by age. For each feature, fit an auto.arima model (R forecast package) across the age-ordered series, selecting the model that minimizes AIC. Retain only features whose selected ARIMA order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0), indicating a genuine longitudinal trend rather than random fluctuation. Z-score the auto.arima fitted values of each retained feature across the age axis so trajectories with different amplitudes are comparable, then cluster the z-scored trajectories with k-means using k=3, the biologically informed cluster count matching declining naive, increasing monocyte/cytotoxic, and stable/other aging patterns. Validate by running the same trend selector on synthetic age-ordered series with implanted monotonic and non-monotonic trends and computing recall and precision of recovery.

## Related tools

- **R Shiny** (Interactive web application used to share and explore the immune-aging results, including the longitudinal genomic trends derived from this trajectory normalization/clustering pipeline) — https://immune-aging.jax.org/
- **forecast (auto.arima)** (R package used to fit the best AIC-minimizing ARIMA model per feature and supply the fitted values that are z-scored prior to k-means clustering)

## Examples

```
library(forecast); fit <- auto.arima(age_ordered_series); z <- scale(as.numeric(fitted(fit))); kmeans(z, centers = 3)
```

## Evaluation signals

- Every retained feature's selected ARIMA order satisfies D>0 and AR+MA>0; features with pure random-fluctuation fits are excluded.
- After z-scoring, each feature's fitted trajectory has mean ~0 and unit variance across the age axis, making cross-feature amplitudes comparable.
- k-means yields k=3 clusters interpretable as the paper's aging patterns: declining naive, increasing monocyte/cytotoxic, and stable/other.
- Synthetic age-ordered series with implanted monotonic and non-monotonic trends are recovered with reported recall and precision when run through the same auto.arima-based selector.
- Output file trajectory_clustering_results.csv contains both cluster assignments and synthetic-benchmark recall/precision columns.

## Limitations

- k=3 is a biologically informed choice tied to this dataset's major aging patterns; other datasets may need a different cluster count.
- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references.
- Not all sex-biased differences in older adults are attributable to changes in cell composition, so trajectory clusters may reflect mixed cell-composition and cell-intrinsic effects.
- Conservation of the aging signatures across ethnicities/populations requires further investigation.

## Evidence

- [methods] Filter the fitted features, retaining only those whose selected ARIMA order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0): [evidence span withheld — anchor status: not_found]
- [methods] Cluster the z-scored fitted trajectories with k-means using k=3, the biologically informed cluster count corresponding to the major aging trajectory patterns: [evidence span withheld — anchor status: not_found]
- [abstract] These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions.: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
