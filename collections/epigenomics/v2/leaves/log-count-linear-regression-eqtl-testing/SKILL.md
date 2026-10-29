---
name: log-count-linear-regression-eqtl-testing
description: Use when estimating eQTL detection power for low-expressed genes (pseudobulk
  mean < 5) where scPower's analytic eQTL power is inaccurate, or when validating
  analytic power with empirical simulation.
when_to_use_negative:
- Gene pseudobulk mean counts are high enough (mean >= 5) for the analytic linear-model
  eQTL power approximation to be accurate — use scPower's analytic power functions
  directly
- The goal is to scan many experimental design options for optimization — scPower's
  analytic method is designed for this; simulation-based runs are too slow
- Data sets with hundreds of samples are required (typical eQTL study sizes) — simulation
  becomes impractical in runtime and memory
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package supplying the expression probability model, NB mean/dispersion priors,
    analytic eQTL power, and the self-implemented simulation method for small-mean
    genes
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which scPower is implemented and the lm() log-count regressions
    are fitted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:c999108ebd40c717b2d1d1134dae220ff1cefb6c41a990bec0d95778214f061a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# log-count linear regression eQTL testing

## Summary

A simulation-based eQTL power estimation approach for genes whose pseudobulk mean expression (mean < 5) is too low for scPower's analytic linear-model approximation, using repeated negative binomial count simulation followed by linear regression of log-transformed counts on genotype.

## When to use

Use when estimating eQTL detection power for low-expressed genes (pseudobulk mean < 5) where scPower's analytic eQTL power is inaccurate, or when validating analytic power with empirical simulation.

## When NOT to use

- Gene pseudobulk mean counts are high enough (mean >= 5) for the analytic linear-model eQTL power approximation to be accurate — use scPower's analytic power functions directly
- The goal is to scan many experimental design options for optimization — scPower's analytic method is designed for this; simulation-based runs are too slow
- Data sets with hundreds of samples are required (typical eQTL study sizes) — simulation becomes impractical in runtime and memory

## Inputs

- scPower package priors: eqtl.ref.study (Blueprint Monocytes eQTL effect sizes), read.umi.fit (e.g. '10X_PBMC_1'), gamma.mixed.fits (mean expression distributions), disp.fun.param (dispersion function parameters)
- Target gene list with eQTL effect sizes (R2) and pseudobulk mean counts (genes with mean < 5)
- Experimental design parameters: sample size, cells per individual, sequencing depth, significance (multiple-testing-corrected) threshold

## Outputs

- Empirical power per gene and design setting from B=100 simulation repeats
- Comparison table of empirical vs analytic scPower power, e.g. written to simulation_vs_analytic_power.csv

## How to apply

Load scPower in R and attach the package priors needed for eQTL power: eqtl.ref.study (Blueprint Monocytes), read.umi.fit ('10X_PBMC_1'), gamma.mixed.fits, and disp.fun.param. Simulate SNP genotypes under Hardy-Weinberg equilibrium, then for each eQTL gene draw negative binomial pseudobulk counts from scPower's mean/dispersion priors, numerically optimizing the dispersion so the simulated data match the target R2. In each of B=100 repeats, fit a linear regression of log-transformed counts on genotype and call a gene significant at the multiple-testing-corrected threshold to obtain empirical power. Compare the resulting empirical power to scPower's analytic power (overall detection power = expression probability × eQTL power) across design settings; agreement within simulation uncertainty validates both.

## Related tools

- **scPower** (R package supplying the expression probability model, NB mean/dispersion priors, analytic eQTL power, and the self-implemented simulation method for small-mean genes) — https://github.com/heiniglab/scPower
- **R** (Language in which scPower is implemented and the lm() log-count regressions are fitted)

## Examples

```
library(scPower); data(eqtl.ref.study); data(read.umi.fit); data(gamma.mixed.fits); data(disp.fun.param); fit <- lm(log(counts) ~ genotype)
```

## Evaluation signals

- simulation_vs_analytic_power.csv exists with one row per gene/design setting
- All power values fall within [0, 1]
- Empirical power agrees with analytic scPower power within expected simulation uncertainty across settings
- Each simulated gene's counts reproduce the target R2 under the numerically optimized dispersion parameter
- Exactly B=100 simulation repeats are fitted per gene and genotypes follow Hardy-Weinberg proportions

## Limitations

- Simulation-based power analysis has long runtimes and high memory requirements, making it unsuitable to evaluate the large number of experimental designs needed to optimize parameter combinations
- Larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate
- Data driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the sample size of the pilot data sets

## Evidence

- [other] For small-mean genes, scPower uses a self-implemented simulation method instead of the analytic eQTL power: [evidence span withheld — anchor status: not_found]
- [readme] scPower R package purpose: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq"
- [methods] Overall detection power definition: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [methods] Negative binomial parameter estimation: "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
