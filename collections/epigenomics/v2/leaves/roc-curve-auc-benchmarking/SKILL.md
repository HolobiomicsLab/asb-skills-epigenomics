---
name: roc-curve-auc-benchmarking
description: Use when comparing differential occupancy / peak-calling methods on a
  dosage-responsive ChIP-seq benchmark with a gold-standard positive set defined at
  a signal quantile, and you need ROC curves and AUC-vs-cutoff comparisons.
when_to_use_negative:
- Input is not a ChIP-seq differential occupancy setting with a defined gold-standard
  positive set — the benchmark quantile-based truth labels do not transfer to other
  data types
- You need a genome-wide production fit rather than a method comparison — use the
  standard GenoGAM workflow (GenoGAMDataSet → computeSizeFactors → genogam → computeSignificance)
  without ROC/AUC benchmarking
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: Fit GAM on benchmark data (lambda = 4601, theta = 4.51) and produce position-wise
    log p-values for ROC/AUC evaluation
  repo: https://github.com/gstricker/fastGenoGAM
- name: GenoGAM
  role: GenoGAM 1.0 fit on identical data as the reference baseline whose ROC curve
    GenoGAM 2.0 must reproduce
  repo: https://github.com/gstricker/GenoGAM
- name: csaw
  role: Window-based differential occupancy competitor producing p-values for the
    ROC/AUC comparison
- name: DESeq2
  role: Region-based differential expression/occupancy competitor producing p-values
    for the ROC/AUC comparison
- name: R
  role: Implementation environment for GenoGAM, csaw, DESeq2, and the ROC/AUC computation
    scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:9c75203e3b9aa3dc84fc037002ba57ab399cd323513f433ac97b994db0606393
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - DESeq2
  - GenoGAM
  - GenoGAM 2.0
  - R
  - csaw
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# roc-curve-auc-benchmarking

## Summary

Benchmark a differential ChIP-seq occupancy caller (e.g. GenoGAM 2.0) against competitors (csaw, DESeq2, GenoGAM 1.0) by sweeping signal-quantile cutoffs to build ROC curves and AUC-vs-cutoff tables, reproducing the original GenoGAM study benchmark (doi:10.1186/s13059-014-0550-8).

## When to use

Use when comparing differential occupancy / peak-calling methods on a dosage-responsive ChIP-seq benchmark with a gold-standard positive set defined at a signal quantile, and you need ROC curves and AUC-vs-cutoff comparisons.

## When NOT to use

- Input is not a ChIP-seq differential occupancy setting with a defined gold-standard positive set — the benchmark quantile-based truth labels do not transfer to other data types
- You need a genome-wide production fit rather than a method comparison — use the standard GenoGAM workflow (GenoGAMDataSet → computeSizeFactors → genogam → computeSignificance) without ROC/AUC benchmarking

## Inputs

- Dosage-responsive benchmark ChIP-seq dataset from the original GenoGAM study (doi:10.1186/s13059-014-0550-8), e.g. BAM files plus experiment design table
- Per-method p-values from GenoGAM 2.0, GenoGAM 1.0, csaw, and DESeq2 on the same dataset
- Signal quantile cutoff grid (gold standard at 0.15 quantile; sweep up to 0.6)

## Outputs

- ROC curves per method (e.g. roc_curve.png)
- AUC values per signal-quantile cutoff per method (e.g. auc_vs_cutoff.csv)

## How to apply

Load the dosage-responsive benchmark ChIP-seq dataset with GenoGAMDataSet() and normalize with computeSizeFactors(); fit GenoGAM 2.0 via genogam() with fixed hyperparameters (lambda = 4601, theta = 4.51) and extract position-wise p-values with computeSignificance(); repeat identically with GenoGAM 1.0 and run csaw (window-based) and DESeq2 (region-based) on the same data for comparable per-position p-values. Define the gold-standard positive set at the 0.15 quantile of the signal, then sweep signal-quantile cutoffs up to 0.6, computing true-/false-positive rates from each method's p-values to assemble ROC curves and AUC values per cutoff. Correct application is confirmed when the GenoGAM 2.0 ROC curve largely overlaps GenoGAM 1.0's, and GenoGAM outperforms csaw and DESeq2 by roughly 0.03-0.04 AUC points for cutoffs up to 0.6.

## Related tools

- **GenoGAM 2.0** (Fit GAM on benchmark data (lambda = 4601, theta = 4.51) and produce position-wise log p-values for ROC/AUC evaluation) — https://github.com/gstricker/fastGenoGAM
- **GenoGAM** (GenoGAM 1.0 fit on identical data as the reference baseline whose ROC curve GenoGAM 2.0 must reproduce) — https://github.com/gstricker/GenoGAM
- **csaw** (Window-based differential occupancy competitor producing p-values for the ROC/AUC comparison)
- **DESeq2** (Region-based differential expression/occupancy competitor producing p-values for the ROC/AUC comparison)
- **R** (Implementation environment for GenoGAM, csaw, DESeq2, and the ROC/AUC computation scripts)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51); pvals <- pval(computeSignificance(result))
```

## Evaluation signals

- GenoGAM 2.0's ROC curve largely overlaps GenoGAM 1.0's (near-identical AUC curves across the cutoff sweep)
- GenoGAM exceeds csaw and DESeq2 by ~0.03-0.04 AUC points up to a quantile cutoff of 0.6
- Gold-standard positive set is reproducibly defined at the 0.15 signal quantile
- Identical size factors and data are used for all four methods so p-values are comparable
- Outputs roc_curve.png and auc_vs_cutoff.csv are produced with per-cutoff AUC rows for each method

## Limitations

- Fixed hyperparameters (lambda = 4601, theta = 4.51) were used to save time at vignette scale rather than cross-validation estimation
- The differential-occupancy benchmark is limited to quantile cutoffs up to 0.6; GenoGAM's advantage is claimed only in that range
- The fastGenoGAM package is deprecated — use the new Bioconductor GenoGAM release
- Requires R 3.4.1 or higher; software licensed under GPL-2

## Evidence

- [results] up to a quantile cutoff of 0.6 GenoGAM outperforms all competitors, including csaw and DESeq2, by around 0.03-0.04 AUC points: [evidence span withheld — anchor status: not_found]
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [methods] compute position-wise log p-values with `computeSignificance()`: "compute position-wise log p-values with `computeSignificance()`"
- [readme] ## compute model without parameter estimation to save time in vignette: "## compute model without parameter estimation to save time in vignette
result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] Note: This version is now depricated. Please use the new version of GenoGAM here: https://bioconductor.org/packages/release/bioc/html/GenoGAM.html: "**Note: This version is now depricated. Please use the new version of GenoGAM here: https://bioconductor.org/packages/release/bioc/html/GenoGAM.html**"
