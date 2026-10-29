---
name: feature-trend-filtering
description: Use when a normalized feature-by-donor abundance matrix (peaks/genes
  vs. adult donors aged 22–93) must be scanned for features whose abundance changes
  systematically with donor age, separating true longitudinal trends from random fluctuation.
when_to_use_negative:
- The donor columns are cross-sectional with no meaningful age ordering or the cohort
  spans too narrow an age range for chronological trends to be estimable.
- The goal is differential abundance between two pre-defined groups rather than continuous
  age-associated trajectories — a differential analysis pipeline is the wrong trend
  filter here.
- Only raw (unnormalized) counts are available and no normalization step can be applied
  first, since auto.arima on unnormalized feature-by-donor values is not what the
  article's workflow assumes.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R Shiny
  role: Platform through which the resulting trajectory/aging analyses are shared
    and explored (immune-aging.jax.org); the repository's Shiny folder contains its
    code.
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
- name: R (forecast package / auto.arima)
  role: Fits the best-fitting ARIMA model per feature across the age-ordered series
    and supplies the AIC-minimizing order used by the trend filter.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:5dc45e5162d2bdd5bbd422f21f7a76553d7cce9b73cd391cc1430960a6eb3e6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R (forecast package / auto.arima)
  - R Shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# feature-trend-filtering

## Summary

Identifies longitudinal (chronological) genomic trends from age-ordered PBMC ATAC-seq/RNA-seq feature-by-donor matrices by fitting auto.arima models per feature, retaining features whose selected ARIMA order (D>0 plus AR+MA>0) indicates a trend distinct from random fluctuations, then z-scoring fitted trajectories for k-means (k=3) clustering.

## When to use

Use when a normalized feature-by-donor abundance matrix (peaks/genes vs. adult donors aged 22–93) must be scanned for features whose abundance changes systematically with donor age, separating true longitudinal trends from random fluctuation.

## When NOT to use

- The donor columns are cross-sectional with no meaningful age ordering or the cohort spans too narrow an age range for chronological trends to be estimable.
- The goal is differential abundance between two pre-defined groups rather than continuous age-associated trajectories — a differential analysis pipeline is the wrong trend filter here.
- Only raw (unnormalized) counts are available and no normalization step can be applied first, since auto.arima on unnormalized feature-by-donor values is not what the article's workflow assumes.

## Inputs

- Normalized feature-by-donor abundance matrix from PBMC ATAC-seq (peaks x donors)
- Normalized feature-by-donor abundance matrix from PBMC RNA-seq (genes x donors)
- Donor ages (used to order the donor columns chronologically)

## Outputs

- Cluster assignments of trend-selected features into k=3 aging trajectory clusters (trajectory_clustering_results.csv)
- Synthetic-benchmark recall and precision metrics for trend-recovery
- Per-feature selected ARIMA orders and fitted (z-scored) trajectory values

## How to apply

Load the normalized feature-by-donor abundance matrix and order donor columns by age so each feature is a chronological series across adult lifespan (22–93 years in this cohort of 172 healthy adults). For each feature (peak/gene), fit auto.arima from the R forecast package across the age-ordered series and select the model minimizing AIC; the fitted model captures the temporal trajectory rather than raw noisy values. Apply the trend filter: retain a feature only if the selected ARIMA order has a differencing term D>0 and at least one autoregressive or moving-average term (AR+MA>0), which indicates a significant longitudinal trend rather than white noise. Z-score the auto.arima fitted values of retained features across the age axis, then cluster the z-scored trajectories with k-means using k=3, corresponding to the major aging patterns (declining naive, increasing monocyte/cytotoxic, stable/other). Validate the selector by generating synthetic age-ordered series with implanted monotonic and non-monotonic trends, rerunning the selector, and computing recall and precision of trended-feature recovery before writing cluster assignments and benchmark metrics to trajectory_clustering_results.csv.

## Related tools

- **R Shiny** (Platform through which the resulting trajectory/aging analyses are shared and explored (immune-aging.jax.org); the repository's Shiny folder contains its code.) — github.com/UcarLab/SexDimorphismNatureCommunications
- **R (forecast package / auto.arima)** (Fits the best-fitting ARIMA model per feature across the age-ordered series and supplies the AIC-minimizing order used by the trend filter.)

## Examples

```
library(forecast); fit <- auto.arima(feature_series_age_ordered); keep <- fit$arma[6] > 0 && (fit$arma[1] + fit$arma[3]) > 0; km <- kmeans(scale(fitted(fit)), centers = 3)
```

## Evaluation signals

- Every retained feature's selected ARIMA order satisfies D>0 AND (AR>0 OR MA>0); features failing either condition are excluded.
- K-means runs with k=3 and the resulting clusters correspond to the expected aging patterns: declining naive, increasing monocyte/cytotoxic, and stable/other.
- Synthetic benchmark with implanted monotonic and non-monotonic trends yields non-zero recall and precision values for trended-feature recovery, recorded alongside cluster assignments.
- Fitted trajectories are z-scored across the age axis before clustering (zero mean, unit variance per feature).
- trajectory_clustering_results.csv contains both cluster assignments and the synthetic recall/precision metrics.

## Limitations

- The trend filter's behavior on features driven by random fluctuations must be empirically calibrated — hence the synthetic monotonic/non-monotonic benchmark, which bounds achievable recall/precision.
- Bulk PBMC feature trends can be confounded by cell-composition shifts; the article notes the bulk annotation approach 'is prone to biases in references' and that some sex/age differences are not attributable to cell composition, so cell-intrinsic trends require caution.
- k=3 is a biologically informed choice tied to the three major trajectory patterns in this cohort (declining naive, increasing monocyte/cytotoxic, stable/other); it may not transfer to other tissues or cohorts.

## Evidence

- [other] Chronological trends in PBMC epigenomic/transcriptomic maps were uncovered by fitting ARIMA models: the best-fitting model was found for each peak/gene as a function of age, and features whose temporal trend was significantly different than random fluctuations were selected.: [evidence span withheld — anchor status: not_found]
- [abstract] These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions.: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry.: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [readme] The code can be found under seperate folders: ... Shiny: Code used for R Shiny app.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
