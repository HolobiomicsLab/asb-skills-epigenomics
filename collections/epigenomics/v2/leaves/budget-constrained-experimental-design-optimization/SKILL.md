---
name: budget-constrained-experimental-design-optimization
description: Use when a single-cell RNA-seq DE or eQTL study has a fixed monetary
  budget and you must choose the optimal trade-off between sample size, cells per
  individual, and sequencing depth to maximize detection power.
when_to_use_negative:
- The user needs power analysis for a simulation-based or single-cell-level DE method
  rather than the pseudobulk approach — simulation tools like muscat/powsimR are infeasible
  here due to 35–48 GB memory needs and long runtimes
- The target sample size greatly exceeds the pilot/reference study sample size, since
  data-driven priors only reliably assess power for samples smaller or roughly equal
  to the pilot
- Continuous cell annotations (e.g. pseudo time) are the unit of analysis — they must
  be discretized before power analysis, and variance QTL power is not supported due
  to lack of effect-size priors
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing optimize.constant.budget and visualize.power.grid for
    budget-constrained design optimization of DE/eQTL scRNA-seq studies
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower package functions are executed
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:284e3bddabfdc57b80950e23dee510b8d1fcd2c2ff142dea460d0c8c321a48e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# budget-constrained-experimental-design-optimization

## Summary

Uses scPower's optimize.constant.budget to sweep cells-per-individual and read-depth grids under a fixed total budget, inferring sample size from the remaining budget and selecting the parameter combination that maximizes overall detection power (powerDetect) for DE or eQTL studies.

## When to use

Use when a single-cell RNA-seq DE or eQTL study has a fixed monetary budget and you must choose the optimal trade-off between sample size, cells per individual, and sequencing depth to maximize detection power.

## When NOT to use

- The user needs power analysis for a simulation-based or single-cell-level DE method rather than the pseudobulk approach — simulation tools like muscat/powsimR are infeasible here due to 35–48 GB memory needs and long runtimes
- The target sample size greatly exceeds the pilot/reference study sample size, since data-driven priors only reliably assess power for samples smaller or roughly equal to the pilot
- Continuous cell annotations (e.g. pseudo time) are the unit of analysis — they must be discretized before power analysis, and variance QTL power is not supported due to lack of effect-size priors

## Inputs

- Total budget in EUR (e.g. 10000 for DE, 30000 for eQTL)
- Cells-per-individual sweep range (nCellsRange, e.g. seq(1000,10000,by=1000))
- Read-depth sweep range (readDepthRange, e.g. seq(10000,50000,by=5000))
- 10X droplet cost model parameters (costKit, costFlowCell, readsPerFlowcell, samplesPerLane, mappingEfficiency)
- scPower example priors: eqtl.ref.study 'Blueprint (Monocytes)', de.ref.study (e.g. Pancreas_alphabeta), gamma.mixed.fits, disp.fun.param, read.umi.fit '10X_PBMC_1'
- Expression cutoff definition (e.g. >3 UMI counts in >50% of samples; Bonferroni for eQTL, FDR for DE)

## Outputs

- Optimal design table (cells per individual, sample size, read depth, powerDetect, computed cost) written e.g. to optimal_design_power_grid.csv
- Power grids over the cells × read-depth parameter combinations
- Power surface visualizations from visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect)

## How to apply

Define a 10X droplet cost model (costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6, samplesPerLane=4, mappingEfficiency=0.8) and load scPower priors such as eqtl.ref.study 'Blueprint (Monocytes)', de.ref.study, gamma.mixed.fits, disp.fun.param, and read.umi.fit '10X_PBMC_1'. Call optimize.constant.budget with totalBudget (e.g. 10,000 EUR for DE, 30,000 EUR for eQTL), sweeping nCellsRange=seq(1000,10000,by=1000) and readDepthRange=seq(10000,50000,by=5000); sample size is inferred from the budget after fixing cells and read depth. Select the grid row maximizing powerDetect: for the DE study this was ~1200 cells in 4 samples at 30,000 reads/cell, and for the eQTL study ~1500 cells in 242 samples at 10,000 reads/cell (Fig. 4). Visualize the resulting power surfaces with visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect) and export the optimal design table. Validation: confirm the selected combination's computed cost does not exceed the budget and its powerDetect is the grid maximum. The rationale is that shallow sequencing of many cells yields higher overall power than deep sequencing of fewer cells, but the exact optimum is use-case specific.

## Related tools

- **scPower** (R package providing optimize.constant.budget and visualize.power.grid for budget-constrained design optimization of DE/eQTL scRNA-seq studies) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower package functions are executed) — https://github.com/heiniglab/scPower

## Examples

```
optimize.constant.budget(totalBudget=10000, nCellsRange=seq(1000,10000,by=1000), readDepthRange=seq(10000,50000,by=5000), samplesPerLane=4, costKit=5600, costFlowCell=14032, readsPerFlowcell=4.1e6, mappingEfficiency=0.8)
```

## Evaluation signals

- The selected optimal row's computed cost (library preparation + sequencing) does not exceed totalBudget
- The selected combination's powerDetect is the maximum over the evaluated cells × read-depth grid
- For the DE study (10,000 EUR) the optimum matches ~1200 cells in 4 samples at 30,000 reads/cell; for the eQTL study (30,000 EUR) ~1500 cells in 242 samples at 10,000 reads/cell (Fig. 4)
- Power surfaces show the expected trend: shallow sequencing of more cells outperforms deep sequencing of fewer cells at fixed budget
- Sample size in each grid row equals the number of samples afforded by the remaining budget given samplesPerLane=4 and mappingEfficiency=0.8

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger samples identify new significant DEGs with lower effect sizes not captured by the priors
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default assumption
- Optimization is restricted to the parameter grids supplied (e.g. 1000–10000 cells, 10000–50000 reads); the true optimum could lie outside the swept range

## Evidence

- [results] Budget-constrained optimum: "In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells."
- [methods] optimization function: "scPower gives the user the opportunity to select the best parameter combination for a restricted budget."
- [methods] power model: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [methods] cost model: "The experimental costs are the sum of the library preparation cost and the sequencing costs."
- [readme] package purpose: "It enables the user to calculate the power for a given experimental setup and to choose for a restricted budget the optimal combination of experimental parameters which maximizes the power."
