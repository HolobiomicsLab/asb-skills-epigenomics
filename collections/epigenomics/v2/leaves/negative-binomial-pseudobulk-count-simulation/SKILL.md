---
name: negative-binomial-pseudobulk-count-simulation
description: Use when analytic power estimates (e.g. from scPower) must be validated
  against simulated count data, when cross-checking scPower against powsimR/muscat,
  or when a gene's pseudobulk mean is too small (< 5) for the analytic linear-model
  eQTL approximation.
when_to_use_negative:
- When evaluating many experimental design parameter combinations for budget optimization
  — the analytic scPower model should be used instead, since simulation-based studies
  have long runtimes unsuitable for scanning large design spaces
- When sample sizes far exceed the pilot data underlying the priors — data-driven
  priors only allow reliable overall-power assessment for sample sizes smaller or
  roughly equal to the pilot study
- When the goal is power for variance QTL detection — no data-driven effect-size priors
  were available and this was not addressed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: Provides pseudobulk construction (create.pseudobulk), negative binomial parameter
    estimation (nbinom.estimation), gamma mixed fits, and the simulation fallback
    for small-mean eQTL genes against which analytic power is compared
  repo: https://github.com/heiniglab/scPower
- name: powsimR
  role: Simulation-based benchmarking tool generating multi-sample count data from
    CD4 T cell negative binomial parameters over the design grid
- name: muscat
  role: Second simulation-based benchmarking tool over the same grid, with edgeR downstream
    DE analysis
- name: edgeR
  role: Pseudobulk differential expression testing (edgeR-LRT) and evaluation of simulated
    datasets at a 0.05 Bonferroni-corrected threshold
- name: R
  role: Implementation environment for scPower package functions and the simulation
    workflow
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:14ec1bf90a2e7c01c59b1964da10601c32819c53edb90627772658985472fa11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - edgeR
  - muscat
  - powsimR
  - scPower
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# negative-binomial-pseudobulk-count-simulation

## Summary

Simulate multi-sample single-cell/pseudobulk UMI count data under a negative binomial model (using scPower's gene-specific mean and dispersion priors) to empirically validate analytic power estimates for pseudobulk DE and eQTL analysis, at known runtime and memory cost.

## When to use

Use when analytic power estimates (e.g. from scPower) must be validated against simulated count data, when cross-checking scPower against powsimR/muscat, or when a gene's pseudobulk mean is too small (< 5) for the analytic linear-model eQTL approximation.

## When NOT to use

- When evaluating many experimental design parameter combinations for budget optimization — the analytic scPower model should be used instead, since simulation-based studies have long runtimes unsuitable for scanning large design spaces
- When sample sizes far exceed the pilot data underlying the priors — data-driven priors only allow reliable overall-power assessment for sample sizes smaller or roughly equal to the pilot study
- When the goal is power for variance QTL detection — no data-driven effect-size priors were available and this was not addressed

## Inputs

- Raw 2d single-cell UMI/read count matrix with sample and cell-type annotations
- scPower package priors: read.umi.fit (e.g. '10X_PBMC_1'), gamma.mixed.fits, disp.fun.param
- Effect size priors for DE genes (e.g. de.ref.study 'Pancreas_alphabeta') or eQTL priors (eqtl.ref.study from Blueprint Monocytes)
- Design grid specification (number of samples, cells per sample, sequencing depth)
- Reference eQTL gene set with target R2 values and MAFs

## Outputs

- Simulated multi-sample pseudobulk count matrices per design grid point
- Empirical power table (expected expressed genes, DE power, overall detection power) per method (scPower, powsimR, muscat) per grid combination
- Per-tool runtime and peak-memory measurements (e.g. scpower_simulation_comparison.csv)
- Per-gene empirical vs analytic eQTL power table (e.g. simulation_vs_analytic_power.csv) for small-mean genes

## How to apply

Build a 3d pseudobulk matrix from the 2d count matrix with scPower's 'create.pseudobulk', then fit gene-specific negative binomial mean and dispersion parameters with 'nbinom.estimation' and a zero-inflated, left-censored gamma mixed distribution over means ('mixed.gamma.estimation'). Simulate count data over a design grid (e.g. nSamples in {4,8,16}, nCells in {200,1000,3000}) with powsimR and/or muscat, timing each run and recording peak memory (paper: 48 GB for powsimR, 35 GB for muscat at 20 samples x 3000 cells). For small-mean eQTL genes (mean < 5), use scPower's own simulation: draw SNP genotypes under Hardy-Weinberg equilibrium, draw negative binomial pseudobulk counts with dispersion numerically optimized to match the target R2, and repeat B=100 times, calling significance by linear regression of log-transformed counts on genotype at a Bonferroni-corrected 0.05 threshold. Analyse simulated datasets with the pseudobulk edgeR-LRT approach, derive empirical expressed-gene counts, DE power and overall detection power, and compare to the scPower analytic estimates (expression probability x eQTL/DE power).

## Related tools

- **scPower** (Provides pseudobulk construction (create.pseudobulk), negative binomial parameter estimation (nbinom.estimation), gamma mixed fits, and the simulation fallback for small-mean eQTL genes against which analytic power is compared) — https://github.com/heiniglab/scPower
- **powsimR** (Simulation-based benchmarking tool generating multi-sample count data from CD4 T cell negative binomial parameters over the design grid)
- **muscat** (Second simulation-based benchmarking tool over the same grid, with edgeR downstream DE analysis)
- **edgeR** (Pseudobulk differential expression testing (edgeR-LRT) and evaluation of simulated datasets at a 0.05 Bonferroni-corrected threshold)
- **R** (Implementation environment for scPower package functions and the simulation workflow) — https://github.com/heiniglab/scPower

## Examples

```
devtools::install_github("heiniglab/scPower"); library(scPower); runShiny()
```

## Evaluation signals

- Simulation-derived expected expressed genes, DE power, and overall power match scPower analytic estimates within simulation uncertainty across every grid combination
- Output CSV contains one row per gene/design setting with power values in [0,1]
- Peak memory is on the reported scale (48 GB for powsimR, 35 GB for muscat at 20 samples x 3000 cells) and runtime shows orders-of-magnitude difference (days vs <1 minute for scPower)
- Empirical power from B=100 simulation repeats agrees with the analytic scPower eQTL power for small-mean genes across settings
- Significance calls use the 0.05 threshold with Bonferroni correction based on the estimated number of expressed genes

## Limitations

- Simulation-based approaches have long runtimes and high memory demands (35-48 GB at 20 samples x 3000 cells), making them unsuitable for evaluating the large number of designs needed for parameter optimization
- Larger datasets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate
- Actual experiments might result in higher doublet rates and a lower number of usable cells than the default assumptions used in the power model

## Evidence

- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation": "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
- [discussion] A sample size of 20 with 3000 cells per sample required already between 35 GB (muscat) and 48GB (powsimR) in our evaluation.: "A sample size of 20 with 3000 cells per sample required already between 35 GB (muscat) and 48GB (powsimR) in our evaluation."
- [intro] In contrast to simulation-based methods, our analytic method can efficiently test a multitude of design options, making it suitable for the optimization of experimental parameters.: "In contrast to simulation-based methods, our analytic method can efficiently test a multitude of design options"
- [readme] scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq.: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
