---
name: sequencing-cost-modeling-droplet-platforms
description: Use when planning a multi-sample single cell DE or eQTL study under a
  fixed budget and you must decide the trade-off between cells per individual, sequencing
  depth (reads/cell), and number of samples on a droplet (10X) platform.
when_to_use_negative:
- The input is already collected single cell count data requiring pseudobulk construction
  (create.pseudobulk, nbinom.estimation) rather than prospective experimental design
- The study uses a non-droplet or non-10X platform whose kit/flowcell cost model and
  read.umi.fit differ from the 10X parameters used here
- Power analysis for variance QTLs or continuous annotations such as pseudo time is
  needed — the paper did not address variance QTL power due to lack of effect size
  priors, and continuous annotations must be discretized first
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing optimize.constant.budget, visualize.power.grid, and example
    priors to model droplet-platform costs and maximize powerDetect under a restricted
    budget
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower package and its optimization functions
    are executed
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:ba22a3f6da9fb9773121cb62a8542e33c13333dd35155003587ac11f20f0dc9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sequencing-cost-modeling-droplet-platforms

## Summary

Model 10X droplet scRNA-seq experimental costs (library prep plus sequencing) and, for a restricted budget, sweep cells-per-individual and read-depth combinations with scPower's optimize.constant.budget to select the design maximizing overall detection power, with sample size inferred from the remaining budget.

## When to use

Use when planning a multi-sample single cell DE or eQTL study under a fixed budget and you must decide the trade-off between cells per individual, sequencing depth (reads/cell), and number of samples on a droplet (10X) platform.

## When NOT to use

- The input is already collected single cell count data requiring pseudobulk construction (create.pseudobulk, nbinom.estimation) rather than prospective experimental design
- The study uses a non-droplet or non-10X platform whose kit/flowcell cost model and read.umi.fit differ from the 10X parameters used here
- Power analysis for variance QTLs or continuous annotations such as pseudo time is needed — the paper did not address variance QTL power due to lack of effect size priors, and continuous annotations must be discretized first

## Inputs

- Experimental budget in EUR (e.g. totalBudget=10000 for DE, 30000 for eQTL)
- Droplet platform cost parameters (costKit, costFlowCell, readsPerFlowcell, samplesPerLane, mappingEfficiency)
- scPower package priors: eqtl.ref.study ('Blueprint (Monocytes)'), de.ref.study, gamma.mixed.fits, disp.fun.param, read.umi.fit ('10X_PBMC_1')
- Parameter grids: nCellsRange and readDepthRange vectors
- Expression definition thresholds (e.g. >3 UMI counts in >50% of samples) and multiple-testing setting (Bonferroni for eQTL, FDR for DE)

## Outputs

- Optimal design table (cells per individual, sample size, read depth, powerDetect, computed cost) per study
- Power grids over cells-per-individual x read-depth combinations
- Power surface plots from visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect)
- CSV export of optimal design and power grids (e.g. optimal_design_power_grid.csv)

## How to apply

Define the droplet cost model with the 10X parameters: costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6, samplesPerLane=4, and mappingEfficiency=0.8, so total cost is the sum of library preparation and sequencing costs. Load scPower example priors (eqtl.ref.study 'Blueprint (Monocytes)', de.ref.study, gamma.mixed.fits, disp.fun.param, read.umi.fit '10X_PBMC_1') and run optimize.constant.budget with totalBudget (e.g. 10,000 EUR for DE, 30,000 EUR for eQTL) while sweeping nCellsRange (e.g. seq(1000,10000,by=1000)) and readDepthRange (e.g. seq(10000,50000,by=5000)); the sample size is inferred from the remaining budget. For each candidate combination scPower computes powerDetect as the product of expression probability and eQTL/DE power; select the row maximizing powerDetect, checking its computed cost does not exceed the budget. Expected optima from the paper: DE ~1200 cells in 4 samples at 30,000 reads/cell; eQTL ~1500 cells in 242 samples at 10,000 reads/cell (Fig. 4). Visualize the power surfaces with visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect) and export the optimal design plus power grids (e.g. optimal_design_power_grid.csv).

## Related tools

- **scPower** (R package providing optimize.constant.budget, visualize.power.grid, and example priors to model droplet-platform costs and maximize powerDetect under a restricted budget) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower package and its optimization functions are executed) — https://github.com/heiniglab/scPower

## Examples

```
optimize.constant.budget(nCellsRange=seq(1000,10000,by=1000), readDepthRange=seq(10000,50000,by=5000), totalBudget=10000, costKit=5600, costFlowCell=14032, readsPerFlowcell=4.1e6, samplesPerLane=4, mappingEfficiency=0.8, expr.definition=expr.definition.scPower, exon.intron.counts=exons.introns.EUR, sig.level=0.05/nGenes, indep.variables=2, ref.study=eqtl.ref.study, ref.study.name="Blueprint (Monocytes)", gamma.mixed.fits=gamma.mixed.fits, disp.fun.param=disp.fun.param, read.umi.fit=read.umi.fit$"10X_PBMC_1")
```

## Evaluation signals

- Selected combination's computed cost does not exceed the specified totalBudget under the 10X cost model (costKit + costFlowCell + sequencing with mappingEfficiency=0.8)
- Selected powerDetect equals the maximum over the evaluated nCellsRange x readDepthRange grid
- Optimum reproduces the paper's Fig. 4 values: DE (10,000 EUR) ~1200 cells in 4 samples at 30,000 reads/cell; eQTL (30,000 EUR) ~1500 cells in 242 samples at 10,000 reads/cell
- Sample size for each grid row is consistent with the budget remainder given samplesPerLane=4 and readsPerFlowcell=4.1e6
- Output CSV contains the optimal design row plus full power grids with columns for totalCells, readDepth, sampleSize, and powerDetect

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger sample sizes may identify new significant DEGs with lower effect sizes not captured in the power computation
- Actual experiments might result in higher doublet rates and lower number of usable cells than the default assumption modelled from the 10X Genomics user guide V3
- Simulation-based alternatives (muscat, powsimR) require long runtimes and high memory (35–48 GB for 20 samples x 3000 cells), making this analytic approach preferable for large parameter sweeps but dependent on prior quality

## Evidence

- [methods] scPower gives the user the opportunity to select the best parameter combination for a restricted budget.: "scPower gives the user the opportunity to select the best parameter combination for a restricted budget."
- [methods] Costs are library prep plus sequencing: "The experimental costs are the sum of the library preparation cost and the sequencing costs."
- [abstract] Shallow sequencing beats deep sequencing: "In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells."
- [readme] scPower is an R package for budget-constrained design: "It enables the user to calculate the power for a given experimental setup and to choose for a restricted budget the optimal combination of experimental parameters which maximizes the power."
- [methods] Multiplet rate modelled from 10X guide: "We modelled the multiplet rate linear increasing with the number of cells loaded per lane, using data from the 10X Genomics user guide V3"
