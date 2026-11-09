---
name: signal-quantile-gold-standard-definition
description: 'Use when benchmarking differential-occupancy peak/differential-region
  callers on ChIP-Seq data without an external truth set: thresholds defined by signal
  quantiles (0.15 positive set, cutoffs to 0.6) provide a reproducible gold standard.'
when_to_use_negative:
- An experimentally validated truth set of differential sites already exists — prefer
  that over signal-quantile proxies
- The comparison is peak calling (occupancy detection) rather than differential occupancy
  between conditions
- The dataset lacks a graded dosage/condition design needed to define a signal-ordered
  positive set
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: GenoGAM 2.0
  role: Produces position-wise log p-values (computeSignificance()) on the benchmark
    dataset for ROC/AUC evaluation against the quantile-defined gold standard
  repo: https://github.com/gstricker/fastGenoGAM
- name: GenoGAM
  role: GenoGAM 1.0 baseline re-fit on identical data to verify that GenoGAM 2.0's
    ROC/AUC performance is preserved
  repo: https://github.com/gstricker/GenoGAM
- name: csaw
  role: Window-based differential occupancy competitor producing per-window p-values
    for the same benchmark
- name: DESeq2
  role: Region-based differential expression/occupancy competitor producing per-region
    p-values for the same benchmark
- name: R
  role: Language in which the gold-standard quantile definition, cutoff sweep, and
    ROC/AUC computation are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:d7ce38de11a4ae20246cfde52f8c5ccdc81d2f0b06deb00fa19a2a20ee4032cd
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Signal-quantile gold-standard definition for differential occupancy ROC benchmarks

## Summary

Defines a gold-standard positive set for differential ChIP-Seq occupancy benchmarking by taking positions above the 0.15 signal quantile as true positives, then sweeping signal-quantile cutoffs up to 0.6 to build ROC curves and AUC values for GenoGAM 2.0 vs csaw and DESeq2.

## When to use

Use when benchmarking differential-occupancy peak/differential-region callers on ChIP-Seq data without an external truth set: thresholds defined by signal quantiles (0.15 positive set, cutoffs to 0.6) provide a reproducible gold standard.

## When NOT to use

- An experimentally validated truth set of differential sites already exists — prefer that over signal-quantile proxies
- The comparison is peak calling (occupancy detection) rather than differential occupancy between conditions
- The dataset lacks a graded dosage/condition design needed to define a signal-ordered positive set

## Inputs

- ChIP-Seq dosage-responsive benchmark dataset (from doi:10.1186/s13059-014-0550-8) as BAM files with an experiment design file
- Position-wise log p-values from GenoGAM 2.0 and GenoGAM 1.0 (via computeSignificance())
- Comparable p-values per region/window from csaw and DESeq2
- Per-position ChIP-Seq signal values used for quantile thresholding

## Outputs

- ROC curves per method (roc_curve.png)
- AUC values as a function of signal-quantile cutoff up to 0.6 (auc_vs_cutoff.csv)
- Gold-standard positive set defined at the 0.15 signal quantile

## How to apply

Obtain position-wise p-values for each method on the same dosage-responsive benchmark ChIP-Seq dataset (GenoGAM 2.0 via GenoGAMDataSet()/computeSizeFactors()/genogam()/computeSignificance(), GenoGAM 1.0 re-fit identically, plus window-based csaw and region-based DESeq2). Define the gold-standard positive set as positions at or above the 0.15 quantile of the ChIP-Seq signal. Sweep signal-quantile cutoffs from the 0.15 quantile up to 0.6, computing true-positive and false-positive rates from each method's p-values at each cutoff to build ROC curves and AUC values. Validate by checking that GenoGAM 1.0 and 2.0 ROC curves largely overlap and that GenoGAM outperforms csaw and DESeq2 by approximately 0.03-0.04 AUC points up to the 0.6 cutoff. Write the results as roc_curve.png and auc_vs_cutoff.csv.

## Related tools

- **GenoGAM 2.0** (Produces position-wise log p-values (computeSignificance()) on the benchmark dataset for ROC/AUC evaluation against the quantile-defined gold standard) — https://github.com/gstricker/fastGenoGAM
- **GenoGAM** (GenoGAM 1.0 baseline re-fit on identical data to verify that GenoGAM 2.0's ROC/AUC performance is preserved) — https://github.com/gstricker/GenoGAM
- **csaw** (Window-based differential occupancy competitor producing per-window p-values for the same benchmark)
- **DESeq2** (Region-based differential expression/occupancy competitor producing per-region p-values for the same benchmark)
- **R** (Language in which the gold-standard quantile definition, cutoff sweep, and ROC/AUC computation are implemented)

## Examples

```
gold <- which(signal >= quantile(signal, 0.15)); tp <- sapply(cutoffs, function(q) sum(pval[which(signal >= quantile(signal, q))] <= alpha & names %in% gold))
```

## Evaluation signals

- GenoGAM 1.0 and GenoGAM 2.0 AUC curves are largely overlapping (near-identical)
- Up to a quantile cutoff of 0.6, GenoGAM outperforms csaw and DESeq2 by around 0.03-0.04 AUC points
- ROC curves are computed over a monotone sweep of signal-quantile cutoffs starting at the 0.15 quantile gold standard
- Position-wise p-values were obtained on identical input data for all compared methods (same dataset, same size factors)
- Outputs roc_curve.png and auc_vs_cutoff.csv exist and contain one AUC value per method per cutoff

## Limitations

- The gold standard is a signal-quantile proxy rather than experimentally validated truth; benchmark validity holds only up to the 0.6 quantile cutoff, beyond which advantage disappears
- GenoGAM 2.0 benchmark used fixed hyperparameters (lambda = 4601, theta = 4.51) tuned for vignette-scale data rather than cross-validated values
- Comparison methods differ in granularity (csaw window-based, DESeq2 region-based, GenoGAM position-wise), affecting comparability of p-value sets

## Evidence

- [other] Gold-standard positives at 0.15 quantile and AUC advantage: [evidence span withheld — anchor status: not_found]
- [readme] Position-wise p-value computation: "compute position-wise log p-values with `computeSignificance()`"
- [intro] GenoGAM different occupancy modeling: "the quantities of interest are often the differential protein occupancies between experiments and controls, or between two genetic backgrounds, or between two treatments,"
- [readme] Fixed hyperparameters for vignette-scale fitting: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [methods] Size factor computation prerequisite: "Computing size factors with `computeSizeFactors()`"
