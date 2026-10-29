---
name: simulation-based-single-cell-power-analysis
description: Use when you need to validate an analytic power-analysis method for multi-sample
  scRNA-seq DE studies against ground-truth simulations, or when you must quantify
  the runtime/memory cost of simulation-based power tools.
when_to_use_negative:
- The goal is rapid power evaluation across many design options — simulation-based
  tools have long runtimes and are unsuitable for optimizing parameter combinations;
  use the analytic method instead
- The target design involves hundreds of samples (e.g. eQTL-scale studies) — such
  data sets will be very difficult to simulate with simulation-based tools
- Sample sizes far exceed the pilot data underlying the priors — data driven priors
  only allow reliable power assessment for sample sizes smaller or roughly equal to
  the pilot study
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scPower
  role: Provides the analytic power estimates (expression probability, DE power, overall
    detection power) used as the reference against which simulations are compared
  repo: https://github.com/heiniglab/scPower
- name: powsimR
  role: Simulates multi-sample single cell count data from negative binomial gene
    parameters over the sample/cell grid for simulation-based power estimation
- name: muscat
  role: Generates alternative multi-sample single cell simulations over the same grid
    for cross-tool agreement and memory benchmarking
- name: edgeR
  role: Performs pseudobulk DE testing (edgeR-LRT) and downstream evaluation on all
    simulated datasets to derive empirical power
- name: R
  role: Implementation environment for the simulation benchmark and comparison-table
    generation
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:45dd72325848d131086be40fba8f8cd9236dd7cedd85df14f4512a0a620e7e75
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# simulation-based-single-cell-power-analysis

## Summary

Benchmark simulation-based power estimation for multi-sample single cell differential expression by simulating count data with powsimR and muscat over a sample-size/cell-number grid and comparing expected expressed genes, DE power, and overall detection power against scPower's analytic estimates, while recording runtime and memory cost.

## When to use

Use when you need to validate an analytic power-analysis method for multi-sample scRNA-seq DE studies against ground-truth simulations, or when you must quantify the runtime/memory cost of simulation-based power tools.

## When NOT to use

- The goal is rapid power evaluation across many design options — simulation-based tools have long runtimes and are unsuitable for optimizing parameter combinations; use the analytic method instead
- The target design involves hundreds of samples (e.g. eQTL-scale studies) — such data sets will be very difficult to simulate with simulation-based tools
- Sample sizes far exceed the pilot data underlying the priors — data driven priors only allow reliable power assessment for sample sizes smaller or roughly equal to the pilot study

## Inputs

- scPower expression priors: read.umi.fit (e.g. '10X_PBMC_1'), gamma.mixed.fits, disp.fun.param
- Effect size priors for DE genes (e.g. scPower::de.ref.study 'Pancreas_alphabeta', 200 DE genes)
- Cell-type-specific negative binomial parameters (per-gene mean and dispersion) for simulation
- Experimental design grid: nSamples in {4,8,16} x nCells in {200,1000,3000}
- Simulated multi-sample single cell UMI count matrices from powsimR and muscat

## Outputs

- scpower_simulation_comparison.csv with expression probability, DE power, and overall detection power for scPower, powsimR, and muscat at every grid combination
- Per-tool runtime (scPower <1 min vs powsimR/muscat ~days) and peak memory figures
- Per-tool DE call sets from edgeR-LRT pseudobulk analysis with Bonferroni-corrected p < 0.05

## How to apply

Load scPower expression priors (read.umi.fit '10X_PBMC_1', gamma.mixed.fits, disp.fun.param) and effect size priors for 200 DE genes (e.g. ref.study 'Pancreas_alphabeta'), and compute analytic expression probability, DE power, and overall detection power over a grid of nSamples in {4,8,16} x nCells in {200,1000,3000}. Independently simulate multi-sample single cell count data with powsimR and muscat from the same cell-type negative binomial parameters (mean/dispersion estimated per gene via nbinom.estimation), timing each run and recording peak memory. Analyse every simulated dataset with the pseudobulk approach (create.pseudobulk, edgeR-LRT testing) at a 0.05 significance threshold with Bonferroni correction based on the estimated number of expressed genes, then derive expected expressed genes, DE power, and overall detection power per tool, scaling test statistics by the effective sample size ratio where applicable. Merge simulation results with the scPower estimates into one table (scpower_simulation_comparison.csv) annotated with per-tool runtime and memory so agreement and computational cost can be judged together.

## Related tools

- **scPower** (Provides the analytic power estimates (expression probability, DE power, overall detection power) used as the reference against which simulations are compared) — https://github.com/heiniglab/scPower
- **powsimR** (Simulates multi-sample single cell count data from negative binomial gene parameters over the sample/cell grid for simulation-based power estimation)
- **muscat** (Generates alternative multi-sample single cell simulations over the same grid for cross-tool agreement and memory benchmarking)
- **edgeR** (Performs pseudobulk DE testing (edgeR-LRT) and downstream evaluation on all simulated datasets to derive empirical power)
- **R** (Implementation environment for the simulation benchmark and comparison-table generation) — https://github.com/heiniglab/scPower

## Examples

```
devtools::install_github("heiniglab/scPower"); library(scPower); pows <- scPower::estimate.power.de.fitted.gammas(nSamples=8, nCells=1000, readDepth=30000, sig.level=0.05)
```

## Evaluation signals

- Simulation-based estimates of expected expressed genes, DE power, and overall detection power from powsimR and muscat are in reasonable agreement with scPower's analytic values at every grid point
- Comparison table contains all three power metrics for all three tools at every combination of nSamples {4,8,16} x nCells {200,1000,3000}
- Memory usage is consistent with the reported scale: 48 GB (powsimR) and 35 GB (muscat) at 20 samples x 3000 cells, vs few MB for scPower
- Runtime ordering holds: scPower < 1 minute, while powsimR and muscat require on the order of days (8 days vs 3 days in the evaluation)
- DE calls derive from edgeR-LRT pseudobulk analysis with a 0.05 significance threshold and Bonferroni correction based on the estimated number of expressed genes

## Limitations

- Simulation-based power analysis has long runtimes and high memory requirements, making it unsuitable to evaluate the large number of experimental designs needed to optimize parameter combinations
- Larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate
- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger samples may identify new significant DEGs with lower effect sizes not captured in the effect size priors
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than assumed

## Evidence

- [discussion] Power agreement and cost: "A sample size of 20 with 3000 cells per sample required already between 35 GB (muscat) and 48GB (powsimR) in our evaluation."
- [intro] Simulation unsuitable for optimization: "A big disadvantage of simulation-based studies are their long runtimes, which make them unsuitable to evaluate the large number of experimental designs needed to optimize parameter combinations."
- [intro] Analytic method rationale: "In contrast to simulation-based methods, our analytic method can efficiently test a multitude of design options, making it suitable for the optimization of experimental parameters."
- [intro] Pseudobulk DE method: "the “pseudobulk” approach in combination with classical differential gene expression methods such as edgeR44 and limma-voom45 outperforms single cell specific methods"
- [methods] Overall power model: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [readme] Install command: [evidence span withheld — anchor status: external_doc]
