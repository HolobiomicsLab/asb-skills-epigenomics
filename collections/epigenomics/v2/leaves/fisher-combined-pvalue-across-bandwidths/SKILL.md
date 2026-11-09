---
name: fisher-combined-pvalue-across-bandwidths
description: Use when temporal genomic/epigenomic series (e.g., PBMC ATAC-seq/RNA-seq
  features across donor ages 22–93) must be scanned for abrupt age-related change
  points rather than gradual trends, and per-age significance must be robust to window-size
  choice.
when_to_use_negative:
- The goal is characterizing gradual, monotonic aging trends rather than abrupt change
  points — use the upstream ARIMA/k-means trajectory analysis instead
- The data are cross-sectional without a reasonably dense age range; the moving-window
  comparison (e.g., ±5 years around a candidate age) needs enough donors per age window
  to support a MANOVA
- The series are single-cell or sorted-cell data — the paper's breakpoint pipeline
  was run on bulk PBMC annotations and its reference-based annotation approach is
  noted to be prone to biases
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the moving-window MANOVA, LOESS smoothing, Fisher
    combination, and breakpoint pipeline
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
- name: mclust
  role: Model-based clustering of loginvp maxima to call final breakpoint ages
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:3c601c5b3ce815ad63f282387487abed696c91370084c7378c0c98831f20f4bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fisher-combined-pvalue-across-bandwidths

## Summary

A breakpoint-detection method that turns per-age MANOVA p-values from moving-window comparisons of temporal epigenomic profiles into robust breakpoint calls by smoothing with LOESS across 11 bandwidths and combining the bandwidth-specific p-values with Fisher's method before Mclust clustering of loginvp maxima.

## When to use

Use when temporal genomic/epigenomic series (e.g., PBMC ATAC-seq/RNA-seq features across donor ages 22–93) must be scanned for abrupt age-related change points rather than gradual trends, and per-age significance must be robust to window-size choice.

## When NOT to use

- The goal is characterizing gradual, monotonic aging trends rather than abrupt change points — use the upstream ARIMA/k-means trajectory analysis instead
- The data are cross-sectional without a reasonably dense age range; the moving-window comparison (e.g., ±5 years around a candidate age) needs enough donors per age window to support a MANOVA
- The series are single-cell or sorted-cell data — the paper's breakpoint pipeline was run on bulk PBMC annotations and its reference-based annotation approach is noted to be prone to biases

## Inputs

- Temporal peak-cluster series (epigenomic/expression features ordered by donor age, 22–93 years) from upstream ARIMA/k-means trajectory clustering
- Synthetic aging series with implanted breakpoints at ages 40 and 65 (positive control)
- Candidate age grid and moving-window spans w = 10–20 years

## Outputs

- Per-age combined loginvp (log inverse p-value) curves from Fisher combination of 11 LOESS-smoothed bandwidth-specific p-values
- Mclust-called breakpoint ages grouped from loginvp maxima
- detected_breakpoints.csv with the final detected breakpoint ages

## How to apply

Start from the temporal peak-cluster series produced by the upstream ARIMA/k-means trajectory clustering (or synthesize aging series with implanted breakpoints at ages 40 and 65 as a positive control). For each window span w between 10 and 20 years, compare profiles at ages ~5 years before and after each candidate age, reduce each windowed profile to 3 dimensions via PCA, and run MANOVA on the 3 principal components to get a per-age p-value for the age-related shift. Smooth each p-value curve with LOESS across 11 different bandwidths, then combine the 11 bandwidth-specific p-values at each age with Fisher's method and transform to log inverse p-values (loginvp). Group loginvp maxima with Mclust (model-based clustering, R mclust package) to call the final breakpoint ages, and write them to detected_breakpoints.csv. Rationale: combining across bandwidths prevents the breakpoint call from depending on an arbitrary window size, which matters because the paper's first epigenomic spike occurs 'around late-thirties' and the second, sex-divergent spike appears after age 65.

## Related tools

- **R** (Implementation language for the moving-window MANOVA, LOESS smoothing, Fisher combination, and breakpoint pipeline) — https://github.com/UcarLab/SexDimorphismNatureCommunications
- **mclust** (Model-based clustering of loginvp maxima to call final breakpoint ages) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
library(mclust); fit <- Mclust(loginvp_maxima, G = 1:9); write.csv(data.frame(age = candidate_ages[fit$classification == max(fit$classification)]), 'detected_breakpoints.csv')
```

## Evaluation signals

- On the positive-control synthetic series with implanted breakpoints at ages 40 and 65, the first detected breakpoint must fall within 38–42 years and the second within 62–71 years
- Exactly 11 LOESS bandwidth-specific p-value curves should be produced per window span before Fisher combination
- detected_breakpoints.csv is produced and its ages match Mclust groupings of the loginvp maxima
- Detected breakpoints should be consistent with the paper's reported first spike around the late thirties (similar in both sexes) and a second, sex-divergent spike after age 65
- Per-age MANOVA p-values should be computed on exactly 3 PCA-reduced dimensions of each windowed profile

## Limitations

- The pipeline depends on an upstream ARIMA/k-means trajectory clustering to produce the temporal peak-cluster series it consumes
- The bulk PBMC annotation approach feeding these temporal series 'is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references'
- Breakpoint timing/interpretation is cohort- and population-specific; the paper notes that 'sex-specific aging signatures described here might be conserved across ethnicities/populations, which requires further investigation'

## Evidence

- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] First spike around late-thirties: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [abstract] Second spike earlier and stronger in men: "whereas the second spike is earlier and stronger in men."
- [abstract] Post-65 sex divergence: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [readme] Code organization in repo: "The code can be found under seperate folders: 1. FlowData: Code used for flow cytometry data analyses 2. Scripts: Scripts for initial data analyses"
