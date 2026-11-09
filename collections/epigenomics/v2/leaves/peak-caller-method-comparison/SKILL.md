---
name: peak-caller-method-comparison
description: Use when comparing differential ChIP-Seq occupancy methods (e.g. GenoGAM
  vs csaw vs DESeq2) on the same dosage-responsive dataset using ROC/AUC against a
  quantile-defined gold standard.
when_to_use_negative:
- Input is a single-condition ChIP-Seq peak-calling task with no differential-contrast
  design — this skill benchmarks differential occupancy callers, not peak discovery.
- The dataset is too small or lacks a dosage/titration structure to define a meaningful
  signal-quantile gold standard (e.g. the package vignette's first 100kb of yeast
  chrI used only to demonstrate the API).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: Fit the genome-wide GAM on the benchmark dataset and produce position-wise
    log p-values via computeSignificance() for ROC/AUC evaluation
  repo: https://github.com/gstricker/fastGenoGAM
- name: GenoGAM
  role: Original (1.0) implementation refit on identical data to verify GenoGAM 2.0
    reproduces its ROC/AUC performance
  repo: https://github.com/gstricker/GenoGAM
- name: csaw
  role: Window-based differential occupancy competitor providing p-values for the
    ROC comparison
- name: DESeq2
  role: Region-based differential competitor providing p-values for the ROC comparison
- name: R
  role: Implementation language for the benchmark workflow (Bioconductor packages
    GenoGAM, csaw, DESeq2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:bd835bab09641686547e7426869df966a9524bc6d0cbfdc25952231663c2f554
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# peak-caller-method-comparison

## Summary

Benchmark differential ChIP-Seq occupancy callers by building ROC curves and AUC-vs-quantile-cutoff plots from position- or region-wise p-values, as done to show GenoGAM 2.0 matches GenoGAM 1.0 and outperforms csaw and DESeq2 by ~0.03-0.04 AUC up to a 0.6 quantile cutoff.

## When to use

Use when comparing differential ChIP-Seq occupancy methods (e.g. GenoGAM vs csaw vs DESeq2) on the same dosage-responsive dataset using ROC/AUC against a quantile-defined gold standard.

## When NOT to use

- Input is a single-condition ChIP-Seq peak-calling task with no differential-contrast design — this skill benchmarks differential occupancy callers, not peak discovery.
- The dataset is too small or lacks a dosage/titration structure to define a meaningful signal-quantile gold standard (e.g. the package vignette's first 100kb of yeast chrI used only to demonstrate the API).

## Inputs

- Dosage-responsive benchmark ChIP-Seq dataset from the original GenoGAM study (doi:10.1186/s13059-014-0550-8), BAM files plus experiment design file
- Per-method p-values: GenoGAM position-wise log p-values from computeSignificance(), csaw window-level p-values, DESeq2 region-level p-values
- Signal quantile cutoffs from 0.15 (gold standard) up to 0.6

## Outputs

- ROC curves per method (roc_curve.png)
- AUC-vs-cutoff table (auc_vs_cutoff.csv)
- Per-method p-value tracks from GenoGAM 2.0, GenoGAM 1.0, csaw, and DESeq2

## How to apply

Load the dosage-responsive benchmark ChIP-Seq dataset with GenoGAMDataSet() and normalize with computeSizeFactors(). Fit GenoGAM 2.0 with genogam() using fixed hyperparameters lambda = 4601 and theta = 4.51 (vignette-scale setting) and obtain position-wise log p-values via computeSignificance(); repeat identically with GenoGAM 1.0. Run csaw (window-based) and DESeq2 (region-based) on the same data to get comparable p-values per method. Define the gold-standard positive set at the 0.15 signal quantile, sweep signal-quantile cutoffs up to 0.6, and compute true-/false-positive rates from each method's p-values to build ROC curves and AUC values. Judge success by GenoGAM 2.0's ROC curve overlapping GenoGAM 1.0's and GenoGAM exceeding csaw and DESeq2 by roughly 0.03-0.04 AUC points for cutoffs up to 0.6; write roc_curve.png and auc_vs_cutoff.csv.

## Related tools

- **GenoGAM 2.0** (Fit the genome-wide GAM on the benchmark dataset and produce position-wise log p-values via computeSignificance() for ROC/AUC evaluation) — https://github.com/gstricker/fastGenoGAM
- **GenoGAM** (Original (1.0) implementation refit on identical data to verify GenoGAM 2.0 reproduces its ROC/AUC performance) — https://github.com/gstricker/GenoGAM
- **csaw** (Window-based differential occupancy competitor providing p-values for the ROC comparison)
- **DESeq2** (Region-based differential competitor providing p-values for the ROC comparison)
- **R** (Implementation language for the benchmark workflow (Bioconductor packages GenoGAM, csaw, DESeq2))

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51); computeSignificance(result)
```

## Evaluation signals

- GenoGAM 2.0's ROC curve largely overlaps GenoGAM 1.0's on the same benchmark data.
- Up to quantile cutoff 0.6, GenoGAM's AUC exceeds csaw and DESeq2 by approximately 0.03-0.04 AUC points.
- Gold-standard positive set is defined at the 0.15 signal quantile; sweep must cover cutoffs up to 0.6.
- Fits use fixed hyperparameters lambda = 4601 and theta = 4.51; size factors computed with computeSizeFactors() before fitting.
- Outputs roc_curve.png and auc_vs_cutoff.csv are produced with per-method AUC values.

## Limitations

- Fixed hyperparameters (lambda = 4601, theta = 4.51) were used to save time at vignette scale rather than cross-validated estimation, so benchmark runtime benefits may not reflect full data-driven smoothing.
- GenoGAM requires R 3.4.1 or higher and is licensed under GPL-2; fastGenoGAM is deprecated in favor of the Bioconductor GenoGAM release.
- Only the 0-0.6 quantile-cutoff range is claimed (GenoGAM outperforms competitors up to 0.6); behavior beyond that range is not asserted.

## Evidence

- [readme] Fixed hyperparameters used in the benchmark fits: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] Standard GenoGAM workflow steps preceding p-value computation: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] Position-wise p-values are the benchmark statistic: "compute position-wise log p-values with `computeSignificance()`"
- [abstract] Key benchmark finding reproduced by the re-run: "a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version"
- [intro] GenoGAM's advantage over windowing/binning competitors: "An important advantage of GenoGAM over competing methods is that smoothing parameters are objectively estimated from the data by cross-validation, eliminating ad-hoc binning and windowing."
