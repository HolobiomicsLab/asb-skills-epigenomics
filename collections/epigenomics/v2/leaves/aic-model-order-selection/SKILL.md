---
name: aic-model-order-selection
description: Use when you have a feature-by-donor abundance matrix (peaks or genes)
  from chronological samples and need to distinguish true age-driven longitudinal
  trends from noise before clustering trajectories.
when_to_use_negative:
- Data are cross-sectional with no donor age ordering, so no chronological series
  can be fit per feature
- Goal is differential abundance between conditions rather than identifying age-ordered
  temporal trends
- Features have already been clustered into age trajectories and only downstream functional
  enrichment or breakpoint analysis is needed
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: R Shiny
  role: R environment (forecast/auto.arima, k-means) used to fit ARIMA models per
    feature and cluster z-scored fitted trajectories; results are also shared via
    a searchable R Shiny application
  repo: https://immune-aging.jax.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:17fe166203f9d2c2d1881a055f74fad17032cbcb2b29c7bf979a61b7fc13ff34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# aic-model-order-selection

## Summary

Identifies longitudinal age-trends in PBMC ATAC-seq/RNA-seq features by fitting auto.arima models per feature across an age-ordered donor series, selecting the minimum-AIC model, and retaining features whose selected ARIMA order (differencing plus AR/MA terms) indicates a genuine temporal trend rather than random fluctuation.

## When to use

Use when you have a feature-by-donor abundance matrix (peaks or genes) from chronological samples and need to distinguish true age-driven longitudinal trends from noise before clustering trajectories.

## When NOT to use

- Data are cross-sectional with no donor age ordering, so no chronological series can be fit per feature
- Goal is differential abundance between conditions rather than identifying age-ordered temporal trends
- Features have already been clustered into age trajectories and only downstream functional enrichment or breakpoint analysis is needed

## Inputs

- Normalized feature-by-donor abundance matrix from PBMC ATAC-seq or RNA-seq data (172 healthy adults aged 22-93), with donor age metadata

## Outputs

- trajectory_clustering_results.csv with per-feature cluster assignments (k=3) and synthetic-benchmark recall/precision metrics
- Per-feature selected ARIMA orders (p, d, q) and minimum-AIC fitted values

## How to apply

Order the normalized feature-by-donor matrix columns by donor age, then for each feature fit auto.arima (R forecast package) across the age-ordered series and select the model minimizing AIC. Filter the fitted features, keeping only those whose selected ARIMA order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0), since these orders indicate a longitudinal trend beyond random fluctuations. Z-score the auto.arima fitted values across the age axis so trajectories are comparable, then cluster them with k-means using k=3, chosen biologically to capture declining naive T cell, increasing monocyte/cytotoxic, and stable/other aging trajectory patterns. Validate the selector by generating synthetic age-ordered series with implanted monotonic and non-monotonic trends and computing recall and precision of trend recovery; write cluster assignments and benchmark metrics to trajectory_clustering_results.csv.

## Related tools

- **R Shiny** (R environment (forecast/auto.arima, k-means) used to fit ARIMA models per feature and cluster z-scored fitted trajectories; results are also shared via a searchable R Shiny application) — https://immune-aging.jax.org/

## Examples

```
Rscript trajectory_clustering.R  # auto.arima(fit, xreg=age_order); select min-AIC; keep D>0 & (AR+MA)>0; scale(fitted); kmeans(z, centers=3) -> trajectory_clustering_results.csv
```

## Evaluation signals

- Every retained feature's selected ARIMA order satisfies the filter: differencing term D>0 AND at least one autoregressive or moving-average term (AR+MA>0)
- For each feature the reported model is the minimum-AIC auto.arima fit across the age-ordered series
- k-means clustering of z-scored fitted values uses k=3, and cluster assignments land in trajectory_clustering_results.csv
- Synthetic benchmark with implanted monotonic and non-monotonic trends recovers trended features with acceptable recall and precision
- Recovered clusters correspond to the expected aging trajectory patterns: declining naive, increasing monocyte/cytotoxic, and stable/other

## Limitations

- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Changes in feature trajectories may reflect shifts in cell composition rather than cell-intrinsic changes; e.g., monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic effects require further study
- The k=3 cluster count is biologically informed but arbitrary; different trajectory shapes may be merged or split
- The AIC+ARIMA-order filter identifies trended features but does not establish functional implications of the identified molecules (IL7R, LEF1, TCF7, IL8, IL18)

## Evidence

- [methods] ARIMA trend selection workflow: [evidence span withheld — anchor status: not_found]
- [methods] ARIMA order filter: [evidence span withheld — anchor status: not_found]
- [methods] k-means on z-scored fits: [evidence span withheld — anchor status: not_found]
- [intro] Trajectory analysis rationale: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] Biological clusters: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [intro] Shiny app sharing: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
