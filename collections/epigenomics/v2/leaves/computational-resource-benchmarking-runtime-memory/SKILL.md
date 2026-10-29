---
name: computational-resource-benchmarking-runtime-memory
description: Use when validating that a fast analytic estimator matches slower simulation-based
  alternatives and you must quantify the computational cost trade-off (wall-clock
  time and peak memory) alongside accuracy.
when_to_use_negative:
- When the analytic method and the simulation-based alternatives estimate fundamentally
  different quantities — benchmark agreement is only meaningful if the same outputs
  (expected expressed genes, DE power, overall power) are derivable from all tools
- When the target design exceeds the pilot data scale (e.g. hundreds of samples for
  eQTL studies) — the article notes such data sets 'will be very difficult to simulate',
  so the simulation arm of the benchmark is infeasible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: Analytic power estimator providing the fast reference estimates (expression
    probability, DE power, overall detection power) in under a minute and a few MB
    of memory
  repo: https://github.com/heiniglab/scPower
- name: powsimR
  role: Simulation-based power analysis arm; simulates multi-sample single cell counts
    from negative binomial parameters, benchmarked at ~8 days runtime and ~48 GB peak
    memory at 20 samples x 3000 cells
- name: muscat
  role: Simulation-based power analysis arm over the same grid, benchmarked at ~3
    days runtime and ~35 GB peak memory
- name: edgeR
  role: Downstream pseudobulk DE testing (edgeR-LRT) and power evaluation of all simulated
    datasets at a 0.05 Bonferroni-corrected threshold
- name: R
  role: Implementation environment for the benchmarking workflow (scPower is an R
    package; simulations and timing run in R)
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:44da586bd5833a86380a84a0c14f8d021e530c6d9d6cb8cc692a1e4fbfc44beb
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# computational-resource-benchmarking-runtime-memory

## Summary

Benchmarks competing computational methods not only on output agreement (here, power estimates from scPower vs simulation-based powsimR and muscat) but also on runtime and peak memory, exposing the orders-of-magnitude cost gap that motivates analytic approaches.

## When to use

Use when validating that a fast analytic estimator matches slower simulation-based alternatives and you must quantify the computational cost trade-off (wall-clock time and peak memory) alongside accuracy.

## When NOT to use

- When the analytic method and the simulation-based alternatives estimate fundamentally different quantities — benchmark agreement is only meaningful if the same outputs (expected expressed genes, DE power, overall power) are derivable from all tools
- When the target design exceeds the pilot data scale (e.g. hundreds of samples for eQTL studies) — the article notes such data sets 'will be very difficult to simulate', so the simulation arm of the benchmark is infeasible

## Inputs

- Expression priors from scPower CD4 T cell reference fits (read.umi.fit '10X_PBMC_1', gamma.mixed.fits, disp.fun.param)
- Effect size priors for 200 DE genes (de.ref.study, e.g. 'Pancreas_alphabeta')
- Parameter grid of sample sizes (4, 8, 16, up to 20) and cells per sample (200, 1000, 3000)
- Negative binomial per-gene mean/dispersion parameters for simulation-based tools

## Outputs

- Comparison table scpower_simulation_comparison.csv with expression probability, DE power, and overall detection power for scPower, powsimR, and muscat at every grid point
- Per-tool runtime and peak memory figures (e.g. 8 days powsimR vs 3 days muscat vs <1 minute scPower; 48 GB vs 35 GB vs few MB)

## How to apply

Run each method over an identical parameter grid — here nSamples in {4,8,16} crossed with nCells in {200,1000,3000} — with scPower computing analytic estimates of expected expressed genes, DE power, and overall detection power, while powsimR and muscat simulate multi-sample CD4 T cell count data from negative binomial parameters. Time every run and record peak memory usage per tool. Analyse all simulated datasets downstream identically (pseudobulk with edgeR-LRT, 0.05 significance threshold with Bonferroni correction based on the estimated number of expressed genes) so power estimates are comparable. Merge the simulation-derived and analytic estimates into one comparison table (scpower_simulation_comparison.csv) with runtime and memory columns per tool. Judge success by agreement of expression probability, DE power, and overall power across tools, and by memory scaling consistent with the reported 35 GB (muscat) / 48 GB (powsimR) at 20 samples x 3000 cells versus few MB for scPower.

## Related tools

- **scPower** (Analytic power estimator providing the fast reference estimates (expression probability, DE power, overall detection power) in under a minute and a few MB of memory) — https://github.com/heiniglab/scPower
- **powsimR** (Simulation-based power analysis arm; simulates multi-sample single cell counts from negative binomial parameters, benchmarked at ~8 days runtime and ~48 GB peak memory at 20 samples x 3000 cells)
- **muscat** (Simulation-based power analysis arm over the same grid, benchmarked at ~3 days runtime and ~35 GB peak memory)
- **edgeR** (Downstream pseudobulk DE testing (edgeR-LRT) and power evaluation of all simulated datasets at a 0.05 Bonferroni-corrected threshold)
- **R** (Implementation environment for the benchmarking workflow (scPower is an R package; simulations and timing run in R)) — https://github.com/heiniglab/scPower

## Examples

```
devtools::install_github("heiniglab/scPower"); library(scPower)  # then benchmark scPower analytic estimates vs powsimR/muscat simulations across the 4/8/16 sample x 200/1000/3000 cell grid, timing runs and recording peak memory
```

## Evaluation signals

- Comparison table contains expression probability, DE power, and overall detection power for all three tools at every grid combination (4/8/16 samples x 200/1000/3000 cells)
- Simulation-based power estimates agree with scPower analytic values within a reasonable tolerance across all three power metrics
- Runtime ordering matches expectations: analytic scPower (<1 minute) orders of magnitude faster than muscat (~3 days) and powsimR (~8 days)
- Memory usage scales consistently with the paper's report of 35 GB (muscat) and 48 GB (powsimR) at 20 samples x 3000 cells versus a few MB for scPower
- All simulated datasets were analysed with the identical downstream pipeline (edgeR-LRT pseudobulk, 0.05 threshold, Bonferroni correction on estimated expressed genes)

## Limitations

- Simulation-based tools scale poorly: 20 samples with 3000 cells per sample already required 35 GB (muscat) to 48 GB (powsimR), and larger designs with hundreds of samples as needed for eQTL studies 'will be very difficult to simulate'
- The benchmark grid was limited to a CD4 T cell prior with 200 DE genes; agreement may not generalise to other cell types, technologies, or much larger effect sets
- Runtime and memory figures are specific to the evaluation hardware and simulation settings and must be re-measured on new setups

## Evidence

- [discussion] Simulation of 20 samples with 3000 cells per sample required 35 GB (muscat) and 48 GB (powsimR) memory: "A sample size of 20 with 3000 cells per sample required already between 35 GB (muscat) and 48GB (powsimR) in our evaluation."
- [intro] Simulation-based methods have long runtimes unsuitable for evaluating many designs: "A big disadvantage of simulation-based studies are their long runtimes, which make them unsuitable to evaluate the large number of experimental designs needed to optimize parameter combinations."
- [intro] Analytic method efficiently tests many design options unlike simulation: "In contrast to simulation-based methods, our analytic method can efficiently test a multitude of design options, making it suitable for the optimization of experimental parameters."
- [methods] scPower models power as product of expression probability and detection power: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [readme] scPower installable from GitHub for reproducing the benchmark: [evidence span withheld — anchor status: external_doc]
