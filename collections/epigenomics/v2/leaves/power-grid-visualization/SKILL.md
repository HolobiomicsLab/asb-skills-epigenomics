---
name: power-grid-visualization
description: Use when scPower's optimize.constant.budget has produced a grid of designs
  over nCells and readDepth for a fixed budget and you must inspect or communicate
  which combination maximizes powerDetect.
when_to_use_negative:
- The goal is power for a single fixed experimental design rather than a parameter
  sweep — use scPower's single-design power calculation instead of grid visualization.
- The input data are raw scRNA-seq counts requiring prior estimation (create.pseudobulk,
  nbinom.estimation, mixed.gamma.estimation) rather than an already-computed power
  grid.
- The question is choosing expression cutoffs (use scPower's select.cutoffs) rather
  than visualizing the cells/read-depth power surface.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package whose optimize.constant.budget produces the power grid and whose
    visualize.power.grid function renders the power surfaces
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Environment in which the scPower package and its visualization functions are
    executed
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:c08c02f6024941b335549a955928702bde2b16ead98c91a4b7bcdd86a70aaf4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# power-grid-visualization

## Summary

Visualize scPower power grids computed over cells-per-individual and read-depth parameter sweeps, rendering surfaces colored by sample size and overall detection power (powerDetect) so the budget-constrained optimum is directly readable.

## When to use

Use when scPower's optimize.constant.budget has produced a grid of designs over nCells and readDepth for a fixed budget and you must inspect or communicate which combination maximizes powerDetect.

## When NOT to use

- The goal is power for a single fixed experimental design rather than a parameter sweep — use scPower's single-design power calculation instead of grid visualization.
- The input data are raw scRNA-seq counts requiring prior estimation (create.pseudobulk, nbinom.estimation, mixed.gamma.estimation) rather than an already-computed power grid.
- The question is choosing expression cutoffs (use scPower's select.cutoffs) rather than visualizing the cells/read-depth power surface.

## Inputs

- scPower power grid data frame from optimize.constant.budget (columns nCells, readDepth, sampleSize, cost, powerDetect)
- Budget-optimization settings (totalBudget, nCellsRange=seq(1000,10000,by=1000), readDepthRange=seq(10000,50000,by=5000))
- scPower example priors: eqtl.ref.study 'Blueprint (Monocytes)', de.ref.study, gamma.mixed.fits, disp.fun.param, read.umi.fit '10X_PBMC_1'

## Outputs

- Power surface plots (totalCells vs readDepth, filled by sampleSize and powerDetect) for the DE and eQTL studies
- Optimal design table and power grids written to optimal_design_power_grid.csv

## How to apply

After running optimize.constant.budget with the 10X droplet cost model (costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6, samplesPerLane=4, mappingEfficiency=0.8), pass the returned power grid to visualize.power.grid. Plot totalCells versus readDepth with the fill/encoding driven by sampleSize and powerDetect, separately for the DE grid (10,000 EUR, expecting the ~1200 cells / 4 samples / 30,000 reads optimum) and the eQTL grid (30,000 EUR, expecting ~1500 cells / 242 samples / 10,000 reads). Use the surfaces to confirm that the design maximizing powerDetect also fits within budget, and write the optimal design row plus the grids to a table such as optimal_design_power_grid.csv. The visualization rationale follows from the paper's finding that shallow sequencing of many cells often outperforms deep sequencing of fewer cells, which the colored surfaces make visually explicit.

## Related tools

- **scPower** (R package whose optimize.constant.budget produces the power grid and whose visualize.power.grid function renders the power surfaces) — https://github.com/heiniglab/scPower
- **R** (Environment in which the scPower package and its visualization functions are executed) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); visualize.power.grid(de.power.grid, colorRows="sampleSize", colorValues="powerDetect")
```

## Evaluation signals

- The selected optimal design's computed cost does not exceed the total budget (10,000 EUR for DE, 30,000 EUR for eQTL).
- The optimal row's powerDetect equals the maximum powerDetect over the evaluated grid.
- For the DE study the visualization surfaces an optimum near 1200 cells in 4 samples at 30,000 reads/cell; for the eQTL study near 1500 cells in 242 samples at 10,000 reads/cell (Fig. 4 expectations).
- Plots encode exactly totalCells vs readDepth with sampleSize and powerDetect as the colored dimensions.
- The output file optimal_design_power_grid.csv contains both the optimal design row and the full power grids.

## Limitations

- Optimum is only over the evaluated grid; the true continuous optimum may fall outside nCellsRange/readDepthRange choices.
- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets, so the visualized power may be optimistic for very large sample sizes.
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default assumption, so the visualized optimum is model-based, not empirical.

## Evidence

- [methods] scPower gives the user the opportunity to select the best parameter combination for a restricted budget.: "scPower gives the user the opportunity to select the best parameter combination for a restricted budget."
- [other] Visualize the power surfaces with visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect) and write the optimal design table plus power grids to optimal_design_power_grid.csv.: [evidence span withheld — anchor status: not_found]
- [abstract] Shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells.: [evidence span withheld — anchor status: not_found]
- [readme] to choose for a restricted budget the optimal combination of experimental parameters which maximizes the power: "to choose for a restricted budget the optimal combination of experimental parameters which maximizes the power"
