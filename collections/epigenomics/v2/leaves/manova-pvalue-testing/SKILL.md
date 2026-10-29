---
name: manova-pvalue-testing
description: Use when an ordered age/time series (e.g., PBMC ATAC/RNA peak-cluster
  trajectories across ages 22–93) must be scanned for abrupt change points rather
  than gradual trends, and per-age p-values for the before-vs-after difference are
  needed.
when_to_use_negative:
- When the goal is to model gradual longitudinal trends rather than abrupt shifts
  — use the upstream ARIMA/k-means trajectory clustering instead
- When the input lacks an ordered age/time axis or has too few time points to fill
  both windows of a 10–20 year span around a candidate age
- When cell-type-resolved or single-cell breakpoint testing is required — the article
  notes the bulk PBMC annotation approach is prone to biases and future work is needed
  at single-cell resolution
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the moving-window MANOVA, LOESS smoothing, and
    Fisher combination of p-values
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
- name: mclust
  role: Model-based clustering (Mclust) of loginvp maxima to call final breakpoint
    ages
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:8e0ecbc686330032189f947adb02f253488267aef588f0de485b1c19fc04f03c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# manova-pvalue-testing

## Summary

A moving-window MANOVA-based breakpoint test for temporal aging data: at each candidate age, profiles from windows immediately before and after (e.g., ±5 years) are reduced to 3 PCA dimensions and tested for a multivariate age-related shift, yielding per-age p-values that are LOESS-smoothed and Fisher-combined to flag ages of rapid epigenomic change.

## When to use

Use when an ordered age/time series (e.g., PBMC ATAC/RNA peak-cluster trajectories across ages 22–93) must be scanned for abrupt change points rather than gradual trends, and per-age p-values for the before-vs-after difference are needed.

## When NOT to use

- When the goal is to model gradual longitudinal trends rather than abrupt shifts — use the upstream ARIMA/k-means trajectory clustering instead
- When the input lacks an ordered age/time axis or has too few time points to fill both windows of a 10–20 year span around a candidate age
- When cell-type-resolved or single-cell breakpoint testing is required — the article notes the bulk PBMC annotation approach is prone to biases and future work is needed at single-cell resolution

## Inputs

- Temporal peak-cluster trajectory series (from upstream ARIMA/k-means trajectory clustering of PBMC ATAC-seq/RNA-seq data, or synthetic aging series with implanted breakpoints)
- Ordered age axis (e.g., donor ages spanning 22–93 years)
- Window span sweep range (w = 10–20 years) and candidate-age step (windows ±5 years around each candidate age)

## Outputs

- Per-candidate-age MANOVA p-values (per window span)
- LOESS-smoothed p-value curves (11 bandwidths)
- Fisher-combined loginvp (log inverse p-value) curve per age
- detected_breakpoints.csv listing called breakpoint ages

## How to apply

Load the temporal peak-cluster series produced by the upstream ARIMA/k-means trajectory clustering (or generate a synthetic aging series with implanted breakpoints at ages 40 and 65 as a positive control). For each candidate age, compare epigenomic profiles in windows immediately preceding and succeeding it (e.g., 5 years before/after), sweeping window spans w between 10 and 20 years; reduce each windowed profile to 3 dimensions via PCA and apply MANOVA on the 3 principal components in R to obtain a p-value for the age-related shift. Smooth each p-value curve with LOESS across 11 different bandwidths, then combine the 11 bandwidth-specific p-values at each age with Fisher's method and transform to log inverse p-values (loginvp). Group the loginvp maxima with mclust (model-based clustering) to call the final breakpoint ages, and write them to detected_breakpoints.csv. Smaller combined p-values flag ages of rapid epigenomic change, so decision points center on window-span sweep range and the mclust grouping of maxima.

## Related tools

- **R** (Implementation language for the moving-window MANOVA, LOESS smoothing, and Fisher combination of p-values) — github.com/UcarLab/SexDimorphismNatureCommunications
- **mclust** (Model-based clustering (Mclust) of loginvp maxima to call final breakpoint ages) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
pcs <- prcomp(window_profiles)$x[,1:3]; pv <- summary(manova(pcs ~ age_window), test="Wilks")$stats[1,"Pr(>F)"]
```

## Evaluation signals

- On the synthetic positive control with implanted breakpoints at ages 40 and 65, the first detected breakpoint must fall within 38–42 years and the second within 62–71 years
- Exactly 11 LOESS bandwidth-specific p-value curves are produced before Fisher combination
- loginvp values are monotonically related to combined p-values (smaller combined p → larger loginvp), and maxima correspond to flagged rapid-change ages
- detected_breakpoints.csv exists and contains called breakpoint ages consistent with the mclust groupings of loginvp maxima
- Called breakpoints reproduce the article's reported first spike 'around late-thirties' and a second, sex-divergent spike after age 65 when run on the real PBMC trajectory data

## Limitations

- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Whether breakpoints generalize across populations is open: 'sex-specific aging signatures described here might be conserved across ethnicities/populations, which requires further investigation'
- Results are sensitive to the moving-window design (window span w between 10 and 20 years, ±5-year pre/post windows) and to the mclust grouping of loginvp maxima; sparse age coverage can distort window comparisons

## Evidence

- [intro] Breakpoint analysis: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] First spike timing: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [abstract] Second spike timing: "whereas the second spike is earlier and stronger in men."
- [readme] Repo purpose: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
