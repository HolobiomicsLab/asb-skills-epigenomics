---
name: age-breakpoint-detection-in-temporal-omics
description: Use when you have age-ordered omics or feature trajectories (e.g., PBMC
  aging series from ATAC-seq/RNA-seq of adults aged 22-93) and need to detect abrupt
  age windows of rapid epigenomic or transcriptomic change rather than assuming gradual
  accumulation.
when_to_use_negative:
- The data are cross-sectional samples without an ordered age axis, so no before/after
  age windows can be compared
- The goal is cell-type-resolved or single-cell sex-difference testing — the source
  study notes its bulk PBMC annotation approach 'is prone to biases in references'
  and that sex differences have not yet been described at single-cell resolution
- You expect strictly gradual change and only need a regression/trajectory fit; the
  moving-window MANOVA is designed to flag abrupt spikes, not model smooth trends
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation environment for the moving-window MANOVA, PCA reduction to
    3 components, LOESS smoothing, and Fisher's-method combination of p-values
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
- name: mclust
  role: Model-based clustering (Mclust) of loginvp maxima to call the final breakpoint
    ages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:39c6d0bc366a990257b204e54a575fe9dfebd8035b150dce8b29eabbcc80f425
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# age-breakpoint-detection-in-temporal-omics

## Summary

A moving-window breakpoint analysis that tests whether age-related changes in temporal omics series (e.g., PBMC ATAC-seq/RNA-seq aging trajectories) accumulate gradually or spike abruptly at specific ages, using MANOVA on PCA-reduced windows, LOESS-smoothed p-values combined by Fisher's method, and Mclust to call breakpoint ages.

## When to use

Use when you have age-ordered omics or feature trajectories (e.g., PBMC aging series from ATAC-seq/RNA-seq of adults aged 22-93) and need to detect abrupt age windows of rapid epigenomic or transcriptomic change rather than assuming gradual accumulation.

## When NOT to use

- The data are cross-sectional samples without an ordered age axis, so no before/after age windows can be compared
- The goal is cell-type-resolved or single-cell sex-difference testing — the source study notes its bulk PBMC annotation approach 'is prone to biases in references' and that sex differences have not yet been described at single-cell resolution
- You expect strictly gradual change and only need a regression/trajectory fit; the moving-window MANOVA is designed to flag abrupt spikes, not model smooth trends

## Inputs

- Temporal peak-cluster omics series (age-ordered PBMC ATAC-seq/RNA-seq/flow-cytometry-derived trajectories from healthy adults aged 22-93)
- Candidate window span range (w = 10-20 years; e.g., ages 5 years before/after a candidate age)
- Synthetic positive-control series with implanted breakpoints at ages 40 and 65 (optional validation input)

## Outputs

- detected_breakpoints.csv listing called breakpoint ages
- Per-age MANOVA p-values for the 3-PC windowed comparison across tested window spans
- LOESS-smoothed p-value curves across 11 bandwidths and Fisher-combined loginvp curves

## How to apply

Start from the temporal peak-cluster series produced by upstream ARIMA/k-means trajectory clustering (or synthesize aging series with implanted breakpoints at ages 40 and 65 as a positive control). For each candidate window span w between 10 and 20 years, compare profiles at ages ~5 years before and after each candidate age, reduce each windowed profile to 3 dimensions via PCA, and run MANOVA on the 3 principal components to obtain a p-value for the age-related shift. Smooth the p-value curves with LOESS across 11 bandwidths, combine the 11 bandwidth-specific p-values at each age with Fisher's method, and transform to log inverse p-values (loginvp). Finally, group loginvp maxima with Mclust (model-based clustering) to call the final breakpoint ages and write them to detected_breakpoints.csv. The rationale, grounded in the Márquez et al. PBMC study, is that age-related epigenomic changes 'first spike around late-thirties' and diverge again after age 65, so windowed comparison flags these ages of rapid change that a monotonic aging model would miss.

## Related tools

- **R** (Implementation environment for the moving-window MANOVA, PCA reduction to 3 components, LOESS smoothing, and Fisher's-method combination of p-values) — https://github.com/UcarLab/SexDimorphismNatureCommunications
- **mclust** (Model-based clustering (Mclust) of loginvp maxima to call the final breakpoint ages)

## Examples

```
library(mclust); breakpoints <- Mclust(loginvp_maxima); write.csv(breakpoints$classification, "detected_breakpoints.csv")
```

## Evaluation signals

- detected_breakpoints.csv is produced and the first detected breakpoint falls within 38-42 years, matching the reported first spike 'around late-thirties'
- The second detected breakpoint falls within 62-71 years, consistent with increased genomic differences between sexes after age 65
- Positive-control series with implanted breakpoints at ages 40 and 65 are recovered at the correct ages (first 38-42, second 62-71)
- Each age's loginvp value is derived from exactly 11 LOESS bandwidths combined via Fisher's method, and each MANOVA was run on exactly 3 principal components per window
- Mclust returns clusters whose maxima correspond to discrete breakpoint ages rather than a single diffuse group (which would indicate no clear breakpoint)

## Limitations

- The second age-related spike is 'earlier and stronger in men', so pooled (sex-agnostic) breakpoint calling may mask sex-divergent timing; males and females should be examined separately
- The underlying analysis used bulk PBMC profiles, and the source study cautions this annotation approach 'is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references'
- Whether changes are abrupt at the individual level is inferred from cohort-level windows; breakpoints reflect rapid cohort-level change, not necessarily a mechanistic event

## Evidence

- [abstract] First age-related epigenomic spike occurs around the late thirties: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [abstract] Second spike is sex-divergent: "whereas the second spike is earlier and stronger in men."
- [abstract] Genomic differences increase after age 65: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [intro] Breakpoint analysis is part of the computational pipeline: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [readme] Repository contains the analysis code: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
