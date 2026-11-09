---
name: pca-based-dimensionality-reduction-of-windowed-profiles
description: Use when each sample contributes a windowed multi-variable profile over
  age (e.g., epigenomic/ATAC-seq or RNA-seq features in moving windows spanning ~10-20
  years) and a multivariate test such as MANOVA needs a compact, low-dimensional representation
  of each window before testing.
when_to_use_negative:
- The data are not ordered along a temporal/age axis, so there are no before/after
  windows to compare (e.g., a static cross-sectional feature table).
- The windowed profiles are already low-dimensional (fewer than ~4 variables per window),
  making PCA reduction unnecessary before MANOVA.
- The goal is differential analysis between sexes or cell types rather than detecting
  ages of rapid change along the lifespan.
tools:
- name: R
  role: Implementation language for the windowed PCA reduction and the surrounding
    breakpoint pipeline (MANOVA, LOESS smoothing, Fisher combination, Mclust clustering)
- name: mclust
  role: Downstream R package that clusters the loginvp maxima after PCA-reduced windows
    yield per-age p-values; constrains how much resolution the 3-PC reduction must
    retain
- name: R Shiny
  role: Dissemination interface through which the article's aging-analysis results
    (including breakpoint outputs) are shared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:10fffc40cb6ecf24d715fc84117d1eb743a1518bfd4a8fe2c8e8ae3d323f9bd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R Shiny
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# PCA-based dimensionality reduction of windowed profiles

## Summary

Reduces each age-windowed epigenomic profile from the PBMC aging time series to 3 principal components so that MANOVA can test age-related shifts in the breakpoint analysis. It is the intermediate step between moving-window profile extraction and MANOVA p-value computation in the trajectory/breakpoint pipeline of the Sexual-dimorphism in human immune system aging study (Nature Communications, 2020).

## When to use

Use when each sample contributes a windowed multi-variable profile over age (e.g., epigenomic/ATAC-seq or RNA-seq features in moving windows spanning ~10-20 years) and a multivariate test such as MANOVA needs a compact, low-dimensional representation of each window before testing.

## When NOT to use

- The data are not ordered along a temporal/age axis, so there are no before/after windows to compare (e.g., a static cross-sectional feature table).
- The windowed profiles are already low-dimensional (fewer than ~4 variables per window), making PCA reduction unnecessary before MANOVA.
- The goal is differential analysis between sexes or cell types rather than detecting ages of rapid change along the lifespan.

## Inputs

- Temporal peak-cluster epigenomic series from ARIMA/k-means trajectory clustering (PBMC ATAC-seq/RNA-seq features ordered by donor age, 22-93 years)
- Moving-window profiles for candidate ages, with window spans w between 10 and 20 years (e.g., profiles at 5 years before/after each candidate age)
- Optionally, synthetic aging series with implanted breakpoints at ages 40 and 65 (positive control)

## Outputs

- 3-dimensional PCA representation (principal component scores) of each windowed profile
- Per-age PCA score matrices consumed by the MANOVA step of the breakpoint analysis

## How to apply

Take the temporal peak-cluster series produced by the upstream ARIMA/k-means trajectory clustering (or synthetic aging series with implanted breakpoints at ages 40 and 65 as a positive control) and, for each candidate age, assemble the profiles from ages immediately preceding and succeeding it (e.g., 5 years before/after) for each window span w between 10 and 20 years. Reduce each windowed profile to 3 dimensions via PCA (the first three PCs summarize the dominant age-related variance in the windowed epigenomic features). Feed those 3 principal components directly into per-age MANOVA to obtain a p-value for the age-related shift; this dimensionality reduction is what makes the multivariate comparison tractable. Downstream, smooth the p-value curves with LOESS across 11 bandwidths, combine them with Fisher's method into loginvp values, and cluster loginvp maxima with Mclust (mclust in R) to call final breakpoint ages. Rationale: the Márquez et al. PBMC cohort (172 healthy adults, 22-93 years, ATAC-seq/RNA-seq/flow cytometry) exhibits abrupt epigenomic spikes (late-thirties; sex-divergent after 65), so the PCA-3D step must preserve age-related variance that separates these windows rather than global variance.

## Related tools

- **R** (Implementation language for the windowed PCA reduction and the surrounding breakpoint pipeline (MANOVA, LOESS smoothing, Fisher combination, Mclust clustering))
- **mclust** (Downstream R package that clusters the loginvp maxima after PCA-reduced windows yield per-age p-values; constrains how much resolution the 3-PC reduction must retain)
- **R Shiny** (Dissemination interface through which the article's aging-analysis results (including breakpoint outputs) are shared)

## Examples

```
pca <- prcomp(windowed_profile_matrix, center = TRUE, scale. = TRUE); pcs <- pca$x[, 1:3]
```

## Evaluation signals

- On the positive-control synthetic series with implanted breakpoints at ages 40 and 65, the detected breakpoints (called after PCA -> MANOVA -> LOESS -> Fisher -> Mclust) must fall within 38-42 years and 62-71 years, matching the reported 'late-thirties' first spike and the post-65 sex-divergent second spike.
- The reduced representation retains exactly 3 principal components per windowed profile, since MANOVA is applied on the 3 principal components.
- Per-age p-values from MANOVA on the 3 PCs should form smooth curves amenable to LOESS smoothing across 11 bandwidths without discontinuities introduced by the PCA step.
- Combined loginvp maxima must form distinct groups that Mclust can cluster into breakpoint calls; degenerate or single-cluster loginvp curves indicate the PCA reduction lost the discriminating age-related signal.
- Results should be reproducible via the repository code (UcarLab/SexDimorphismNatureCommunications) and consistent with outputs browsable in the immune-aging R Shiny application.

## Limitations

- PCA retains only the top 3 components of each windowed profile; age-related shifts captured in lower-variance components may be missed by the subsequent MANOVA.
- The windowed profiles depend on upstream trajectory clustering (ARIMA/k-means peak clusters); errors or noise in that clustering propagate into the PCA-3D representation.
- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references, which can confound the windowed features being reduced.
- The method identifies ages of rapid change but not the mechanism or cell-type-resolved source; sex differences at single-cell resolution and functional implications remain unestablished per the article's discussion.

## Evidence

- [full_text] workflow windowed PCA feeding MANOVA: [evidence span withheld — anchor status: not_found]
- [intro] breakpoint pipelines in the article: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] cohort and data modalities: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] first spike timing: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [readme] repository code location: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
