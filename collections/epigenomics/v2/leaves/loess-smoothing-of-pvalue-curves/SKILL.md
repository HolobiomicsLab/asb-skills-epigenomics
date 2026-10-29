---
name: loess-smoothing-of-pvalue-curves
description: Use when moving-window age-comparison tests (e.g., MANOVA on windowed
  profiles) yield noisy per-age p-value curves that must be smoothed across several
  bandwidths and combined before calling breakpoint ages.
when_to_use_negative:
- Data are not temporally ordered by age (e.g., a case/control or single-time-point
  comparison) — moving-window breakpoint statistics and their smoothing do not apply
- The per-age p-value curve is already smooth and monotone; single-span LOESS or no
  smoothing suffices and the 11-bandwidth Fisher combination would add unnecessary
  complexity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation language for the LOESS smoothing, Fisher's-method combination,
    and downstream breakpoint analysis pipeline
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
- name: mclust
  role: Model-based clustering (Mclust) of the loginvp maxima after smoothing/combination
    to call final breakpoint ages
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:dba1245db92bab7c017b4f2dc5bb4876daabd0b95af825203a186d7c48924427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# loess-smoothing-of-pvalue-curves

## Summary

Non-parametric LOESS smoothing of per-age p-value curves across multiple bandwidths, followed by Fisher's-method combination, to stabilize noisy moving-window breakpoint statistics in temporal epigenomic (PBMC ATAC/RNA) aging data.

## When to use

Use when moving-window age-comparison tests (e.g., MANOVA on windowed profiles) yield noisy per-age p-value curves that must be smoothed across several bandwidths and combined before calling breakpoint ages.

## When NOT to use

- Data are not temporally ordered by age (e.g., a case/control or single-time-point comparison) — moving-window breakpoint statistics and their smoothing do not apply
- The per-age p-value curve is already smooth and monotone; single-span LOESS or no smoothing suffices and the 11-bandwidth Fisher combination would add unnecessary complexity

## Inputs

- Temporal peak-cluster epigenomic series (per-cluster values across donor ages 22–93, derived from PBMC ATAC-seq/RNA-seq of 172 healthy adults) or synthetic aging series with implanted breakpoints at ages 40 and 65
- Per-age raw p-values from MANOVA on the 3 principal components of windows before/after each candidate age, computed for window spans w = 10–20 years

## Outputs

- Bandwidth-specific LOESS-smoothed p-value curves (11 curves, one per bandwidth)
- Per-age combined statistic (Fisher's-method loginvp values)
- detected_breakpoints.csv listing called breakpoint ages (expected: first within 38–42 years, second within 62–71 years)

## How to apply

From the upstream ARIMA/k-means temporal peak-cluster series (PBMC profiles of healthy adults aged 22–93), compute a moving-window p-value at each candidate age by comparing profiles ~5 years before vs. ~5 years after the candidate age, with the MANOVA applied to the first 3 principal components of each windowed profile over window spans w between 10 and 20 years. Smooth the resulting raw p-value curves with LOESS across 11 different bandwidths rather than a single span, so breakpoint timing is not an artifact of one smoothing scale. At each age, combine the 11 bandwidth-specific p-values using Fisher's method and transform to log inverse p-values (loginvp). Feed the loginvp maxima to Mclust (model-based clustering) to call final breakpoint ages, then verify calls against the paper's positive control: synthetic aging series with implanted breakpoints at ages 40 and 65. Validate the workflow by confirming the first detected spike falls within 38–42 years ('around late-thirties') and the second within 62–71 years (sex-divergent spike after 65).

## Related tools

- **R** (Implementation language for the LOESS smoothing, Fisher's-method combination, and downstream breakpoint analysis pipeline) — https://github.com/UcarLab/SexDimorphismNatureCommunications
- **mclust** (Model-based clustering (Mclust) of the loginvp maxima after smoothing/combination to call final breakpoint ages) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
bws <- seq(0.05, 0.5, length.out = 11); smoothed <- lapply(bws, function(s) predict(loess(p_value ~ age, span = s, data = manova_pvals), newdata = grid_ages))
```

## Evaluation signals

- Positive-control check: on synthetic aging series with implanted breakpoints at ages 40 and 65, the first detected breakpoint must lie within 38–42 years and the second within 62–71 years
- First detected spike should match the paper's reported timing 'around late-thirties'; the second should be sex-divergent and occur after age 65
- detected_breakpoints.csv is produced with called breakpoint ages
- Smoothed curves should be qualitatively stable across all 11 bandwidths; if a spike appears under only one bandwidth, it is likely a smoothing artifact
- MANOVA p-values are computed on exactly 3 principal components per windowed profile, as specified by the pipeline

## Limitations

- The breakpoint pipeline depends on the upstream trajectory analysis (ARIMA/k-means clustering of temporal peak clusters); errors there propagate into the p-value curves
- Reported breakpoint ages are calibrated for healthy-adult PBMC data spanning 22–93 years; other tissues, cohorts, or age ranges may shift spike locations
- The repository README describes code organized in FlowData, Scripts, Figures, and Shiny folders with limited documentation, so reproducing exact bandwidth parameters may require inspecting the figure scripts
- The mechanism of the detected epigenomic aging spikes (e.g., relation to CpG-clock entropic decay) is not established by this analysis

## Evidence

- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [methods] epigenomic profiles at ages immediately preceding and succeeding a candidate age (e.g., 5 years before/after) are compared, and a p-value is computed for the difference between the windows: [evidence span withheld — anchor status: not_found]
- [methods] Smooth the resulting p-value curves with LOESS across 11 different bandwidths, then combine the bandwidth-specific p-values at each age using Fisher's method.: [evidence span withheld — anchor status: not_found]
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
