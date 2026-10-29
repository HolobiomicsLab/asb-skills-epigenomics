---
name: analytic-vs-empirical-power-comparison
description: Use when validating or trusting scPower power estimates for low-mean
  eQTL genes, i.e. when pseudobulk mean expression falls below the regime where the
  analytic linear-model approximation holds and simulation is needed as a cross-check.
when_to_use_negative:
- The goal is to power large eQTL cohorts with hundreds of samples — simulation-based
  validation scales poorly, since larger data sets required for eQTL studies are very
  difficult to simulate
- Genes have sufficient pseudobulk mean expression (mean >= 5), where the analytic
  linear-model approximation is already accurate and simulation adds little
- The task is DEG power analysis or experimental-parameter optimization, where the
  analytic method should be used directly instead of simulations
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing the analytic power functions, expression probability model,
    and package priors (eqtl.ref.study, read.umi.fit, gamma.mixed.fits, disp.fun.param)
    used to simulate and compare power
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower package is installed via devtools
    and the simulation/comparison script is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:b01f8f5150d3659d06bc255ed8703be412f56364c267c0e1b2721f8b09e270c0
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# analytic-vs-empirical-power-comparison

## Summary

Benchmark scPower's analytic eQTL power formula against an empirical simulation for genes whose pseudobulk mean expression is too small (mean < 5) for the analytic linear-model approximation to be accurate, using scPower's expression probability model to assign gene means.

## When to use

Use when validating or trusting scPower power estimates for low-mean eQTL genes, i.e. when pseudobulk mean expression falls below the regime where the analytic linear-model approximation holds and simulation is needed as a cross-check.

## When NOT to use

- The goal is to power large eQTL cohorts with hundreds of samples — simulation-based validation scales poorly, since larger data sets required for eQTL studies are very difficult to simulate
- Genes have sufficient pseudobulk mean expression (mean >= 5), where the analytic linear-model approximation is already accurate and simulation adds little
- The task is DEG power analysis or experimental-parameter optimization, where the analytic method should be used directly instead of simulations

## Inputs

- scPower package priors: eqtl.ref.study (Blueprint Monocytes eQTL reference), read.umi.fit ('10X_PBMC_1'), gamma.mixed.fits, disp.fun.param
- Set of eQTL genes with pseudobulk mean expression < 5
- Design parameters (sample size, cells per sample, sequencing depth, target R2, multiple-testing-corrected significance threshold)
- B = 100 simulation repeats per gene/design setting

## Outputs

- simulation_vs_analytic_power.csv with one row per gene/design setting containing empirical and analytic power values
- Per-gene empirical power estimates from 100 linear-regression repeats on simulated negative binomial counts

## How to apply

Load scPower in R and attach the package priors needed for eQTL power: eqtl.ref.study from the Blueprint Monocytes reference study, read.umi.fit '10X_PBMC_1', gamma.mixed.fits, and disp.fun.param. Simulate SNP genotypes under Hardy-Weinberg equilibrium, then for each small-mean eQTL gene draw negative binomial pseudobulk counts using scPower's mean/dispersion priors, numerically optimizing the dispersion so the simulated data match the target R2. In each of B=100 repeats fit a linear regression of log-transformed counts on genotype and call a gene significant at the multiple-testing-corrected threshold to obtain empirical power. Compute the analytic power for the same design — overall detection power being the product of the expression probability of a gene and the eQTL power of a gene — and tabulate empirical vs analytic values per gene/design setting in simulation_vs_analytic_power.csv.

## Related tools

- **scPower** (R package providing the analytic power functions, expression probability model, and package priors (eqtl.ref.study, read.umi.fit, gamma.mixed.fits, disp.fun.param) used to simulate and compare power) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower package is installed via devtools and the simulation/comparison script is executed)

## Examples

```
library(scPower); data(eqtl.ref.study); data(read.umi.fit); data(gamma.mixed.fits); data(disp.fun.param)
```

## Evaluation signals

- simulation_vs_analytic_power.csv exists with exactly one row per gene/design setting
- All empirical and analytic power values fall within [0, 1]
- Empirical power from the 100-repeat simulation agrees with the analytic scPower power within the expected simulation uncertainty across design settings
- Simulated SNP genotypes conform to Hardy-Weinberg equilibrium
- The simulated negative binomial dispersion was numerically optimized so that simulated data match the target R2

## Limitations

- Simulation-based validation has long runtimes, making it unsuitable for evaluating the large number of experimental designs needed to optimize parameter combinations — the analytic method is preferred for design exploration
- Larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate, restricting the empirical comparison to small designs
- The self-implemented simulation is used only for genes whose pseudobulk mean is too small for the analytic linear-model approximation to be accurate

## Evidence

- [methods] overall power as expression probability times eQTL power: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [intro] analytic method favored for design optimization: "In contrast to simulation-based methods, our analytic method can efficiently test a multitude of design options, making it suitable for the optimization of experimental parameters."
- [discussion] simulation infeasible for large eQTL cohorts: "larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate."
- [readme] scPower as R power-analysis package: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
