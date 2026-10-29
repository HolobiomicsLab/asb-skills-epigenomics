---
name: arima-time-series-trend-modeling
description: Use when a feature-by-donor abundance matrix (e.g., ATAC-seq peaks or
  RNA-seq genes across adults aged 22–93) must be scanned for features whose abundance
  changes systematically with donor age rather than fluctuating randomly.
when_to_use_negative:
- The data are cross-sectional cell-type-frequency comparisons or differential analyses
  rather than age-ordered longitudinal series
- The goal is breakpoint detection or functional enrichment, which the paper treats
  as separate computational pipelines from trajectory analysis
- There is no age or time ordering of samples, so ARIMA differencing over chronological
  order is meaningless
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3512
tools:
- name: R Shiny
  role: Interactive platform (https://immune-aging.jax.org/) via which the trajectory/trend
    analysis results are shared and explored
  repo: https://immune-aging.jax.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:2284dac607a0e7f976a4ffb1e8817512765d3d733fdf908af55b58d134d00daf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# arima-time-series-trend-modeling

## Summary

Identifies longitudinal (chronological) genomic trends by fitting auto.arima models to age-ordered PBMC ATAC-seq/RNA-seq feature series, filtering for ARIMA orders with a differencing term plus AR/MA terms, and clustering the z-scored fitted trajectories with k-means (k=3).

## When to use

Use when a feature-by-donor abundance matrix (e.g., ATAC-seq peaks or RNA-seq genes across adults aged 22–93) must be scanned for features whose abundance changes systematically with donor age rather than fluctuating randomly.

## When NOT to use

- The data are cross-sectional cell-type-frequency comparisons or differential analyses rather than age-ordered longitudinal series
- The goal is breakpoint detection or functional enrichment, which the paper treats as separate computational pipelines from trajectory analysis
- There is no age or time ordering of samples, so ARIMA differencing over chronological order is meaningless

## Inputs

- Normalized feature-by-donor abundance matrix (ATAC-seq peaks or RNA-seq genes) with donor metadata including age
- k=3 cluster count (biologically informed: declining naive, increasing monocyte/cytotoxic, stable/other)

## Outputs

- Per-feature selected ARIMA order and trend-retention status
- Z-scored auto.arima fitted trajectories per retained feature
- k-means (k=3) cluster assignments per feature
- Synthetic-benchmark recall and precision metrics written to trajectory_clustering_results.csv

## How to apply

Load the normalized feature-by-donor matrix and order donor columns by age. For each feature (peak/gene), fit an auto.arima model (forecast package, R) across the age-ordered series and select the model minimizing AIC. Retain only features whose selected order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0), indicating a genuine longitudinal trend rather than random fluctuation. Z-score the auto.arima fitted values of retained features along the age axis, then cluster the z-scored trajectories with k-means using k=3, matching the paper's major aging patterns: declining naive, increasing monocyte/cytotoxic, and stable/other. Validate the selector by implanting monotonic and non-monotonic trends into synthetic age-ordered series and computing recall and precision of recovery before writing cluster assignments and benchmark metrics to trajectory_clustering_results.csv.

## Related tools

- **R Shiny** (Interactive platform (https://immune-aging.jax.org/) via which the trajectory/trend analysis results are shared and explored) — https://immune-aging.jax.org/

## Examples

```
fit <- forecast::auto.arima(ts(feature_series_age_ordered)); keep <- fit$arma[6] > 0 && (fit$arma[1] + fit$arma[2]) > 0
```

## Evaluation signals

- Retained features all have selected ARIMA order with D>0 and AR+MA>0
- AIC of the selected auto.arima model is minimal among candidate models per feature
- k-means clusters (k=3) reproduce the expected patterns: declining naive, increasing monocyte/cytotoxic, stable/other
- Synthetic-benchmark recall and precision of trend recovery are computed and reported in trajectory_clustering_results.csv
- Z-scored fitted values are centered (mean 0, sd 1) along the age axis before clustering

## Limitations

- Trend selection depends on ARIMA order properties (D>0, AR+MA>0), so features with subtle trends fitting a purely random-walk or white-noise model may be excluded
- The k=3 cluster count is biologically informed for this PBMC aging dataset and may not transfer to other tissues or cohorts
- The bulk PBMC annotation approach underlying the feature matrix is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Repository provides scripts organized under FlowData, Scripts, Figures, and Shiny folders; the manuscript is a biorxiv preprint version (10.1101/755702)

## Evidence

- [methods] Chronological trends in PBMC epigenomic/transcriptomic maps were uncovered by fitting ARIMA models: the best-fitting model was found for each peak/gene as a function of age: [evidence span withheld — anchor status: not_found]
- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions.: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [readme] The code can be found under seperate folders: 1. FlowData: Code used for flow cytometry data analyses 2. Scripts: Scripts for initial data analyses: "The code can be found under seperate folders: 
1. FlowData: Code used for flow cytometry data analyses
2. Scripts: Scripts for initial data analyses"
