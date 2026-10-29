---
name: mclust-model-based-clustering-of-maxima
description: Use when temporal aging series (e.g., PBMC ATAC-seq/RNA-seq profiles
  across ages 22–93) yield per-age p-value maxima that must be grouped into discrete
  breakpoint ages, with positive-control breakpoints expected near 40 and 65 years.
when_to_use_negative:
- Input is a raw ATAC-seq/RNA-seq count matrix that has not yet been reduced to temporal
  peak-cluster trajectories via the upstream ARIMA/k-means clustering.
- The goal is gradual trend quantification (e.g., epigenetic clock/CpG entropy) rather
  than detecting abrupt change points — the article notes CpG clock mechanisms represent
  'widespread entropic decay', a different aging model.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language for the breakpoint pipeline (moving-window MANOVA,
    LOESS, Fisher combination, mclust grouping)
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
- name: mclust
  role: Model-based clustering (BIC-selected Gaussian mixture) used to group loginvp
    maxima and call final breakpoint ages
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:d3dc65ff42275224d7d2c6a063ef43dfc29f391941a34746884a450e848ac885
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mclust-model-based-clustering-of-maxima

## Summary

Final step of the breakpoint analysis pipeline from Sexual-dimorphism in human immune system aging: Fisher-combined, LOESS-smoothed MANOVA p-value maxima (loginvp) are grouped with mclust model-based clustering to call the ages at which epigenomic change in PBMCs spikes abruptly rather than accumulating gradually.

## When to use

Use when temporal aging series (e.g., PBMC ATAC-seq/RNA-seq profiles across ages 22–93) yield per-age p-value maxima that must be grouped into discrete breakpoint ages, with positive-control breakpoints expected near 40 and 65 years.

## When NOT to use

- Input is a raw ATAC-seq/RNA-seq count matrix that has not yet been reduced to temporal peak-cluster trajectories via the upstream ARIMA/k-means clustering.
- The goal is gradual trend quantification (e.g., epigenetic clock/CpG entropy) rather than detecting abrupt change points — the article notes CpG clock mechanisms represent 'widespread entropic decay', a different aging model.

## Inputs

- Temporal peak-cluster epigenomic series from upstream ARIMA/k-means trajectory clustering of PBMC ATAC-seq/RNA-seq data (ages 22–93)
- Synthetic aging series with implanted breakpoints at ages 40 and 65 (positive control)
- Per-age loginvp (log inverse Fisher-combined p-value) curves from MANOVA + LOESS smoothing across 11 bandwidths

## Outputs

- detected_breakpoints.csv listing called breakpoint ages per peak cluster
- mclust classification of loginvp maxima into breakpoint vs. non-breakpoint groups

## How to apply

Load the temporal peak-cluster series from the upstream ARIMA/k-means trajectory clustering (or synthetic aging series with implanted breakpoints at ages 40 and 65). For each window span w between 10 and 20 years, compare profiles at ages 5 years before/after each candidate age, reduce each windowed profile to 3 dimensions via PCA, and run MANOVA on those 3 principal components to get a per-age p-value for the age-related shift. Smooth the p-value curves with LOESS across 11 different bandwidths, combine the 11 bandwidth-specific p-values at each age with Fisher's method, and transform to log inverse p-values (loginvp). Apply mclust (R, model-based clustering with BIC model selection) to group the loginvp maxima and call final breakpoint ages, writing them to detected_breakpoints.csv.

## Related tools

- **R** (Implementation language for the breakpoint pipeline (moving-window MANOVA, LOESS, Fisher combination, mclust grouping)) — github.com/UcarLab/SexDimorphismNatureCommunications
- **mclust** (Model-based clustering (BIC-selected Gaussian mixture) used to group loginvp maxima and call final breakpoint ages) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
library(mclust); fit <- Mclust(loginvp_maxima); write.csv(data.frame(age=names(fit$classification), cluster=fit$classification), 'detected_breakpoints.csv')
```

## Evaluation signals

- On the positive control (breakpoints implanted at ages 40 and 65), the first detected breakpoint lies within 38–42 years.
- The second detected breakpoint lies within 62–71 years, matching the reported sex-divergent spike after age 65.
- detected_breakpoints.csv is produced with one breakpoint age per temporal peak cluster.
- Mclust fits assign the loginvp maxima to discrete clusters whose member ages coincide with the maxima of the Fisher-combined (loginvp) p-value curve.
- Called breakpoints correspond to ages flagged by small combined p-values across all 11 LOESS bandwidths, not just a single bandwidth.

## Limitations

- Breakpoint calling depends entirely on the upstream temporal peak-cluster series (ARIMA/k-means trajectory clustering); errors there propagate into the loginvp maxima being clustered.
- Results are sensitive to the window-span search range (w = 10–20 years) and the ±5-year window offset; spans outside this range were not validated.
- The method was validated against a positive control with implanted breakpoints (ages 40 and 65) and the reported late-thirties/after-65 spikes; performance on other cohorts or age distributions is unestablished, consistent with the article's caveat that sex-specific aging signatures 'might be conserved across ethnicities/populations, which requires further investigation'.

## Evidence

- [abstract] Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes: "Age-related epigenomic changes first spike around late-thirties with similar timing and magnitude between sexes"
- [abstract] genomic differences between sexes increase after age 65: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
