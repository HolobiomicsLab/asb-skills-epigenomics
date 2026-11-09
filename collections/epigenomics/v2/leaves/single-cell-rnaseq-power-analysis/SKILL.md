---
name: single-cell-rnaseq-power-analysis
description: Use when planning an interindividual single cell RNA-seq differential
  expression or eQTL study and you need to choose sample size, cells per sample, and
  sequencing depth to maximize detection power under a fixed budget.
when_to_use_negative:
- The goal is a single-cell-level (not pseudobulk) DE method evaluation — scPower's
  model is specifically developed for the pseudobulk approach
- Sample sizes far exceed the pilot/reference study used for priors — data-driven
  priors only reliably assess power for sample sizes smaller or roughly equal to the
  pilot data
- The study targets variance QTL detection or continuous annotations like pseudo time
  — the article did not address variance QTLs (no effect size priors) and pseudo time
  would need discretization
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package providing the analytical power model, priors, budget-constrained
    optimization (optimize.constant.budget), and visualization (visualize.power.grid)
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which scPower is implemented and all power/design calculations
    are executed
  repo: https://github.com/heiniglab/scPower
- name: edgeR
  role: Pseudobulk DE method whose performance motivates the negative binomial pseudobulk
    power model used for the DE power term
- name: limma-voom
  role: Pseudobulk DE method cited alongside edgeR as outperforming single-cell-specific
    methods, motivating the analytic pseudobulk model
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:2b770dcf189c017d0d667a1d872c458d17b250435bbd67f336d78985ec32880b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - edgeR
  - limma-voom
  - scPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-rnaseq-power-analysis

## Summary

Analytical power analysis and experimental design optimization for multi-sample single cell RNA-seq DE and eQTL studies using scPower, which multiplies per-gene expression probability by pseudobulk DE/eQTL power and selects the parameter combination (sample size, cells per individual, read depth) maximizing power for a restricted budget.

## When to use

Use when planning an interindividual single cell RNA-seq differential expression or eQTL study and you need to choose sample size, cells per sample, and sequencing depth to maximize detection power under a fixed budget.

## When NOT to use

- The goal is a single-cell-level (not pseudobulk) DE method evaluation — scPower's model is specifically developed for the pseudobulk approach
- Sample sizes far exceed the pilot/reference study used for priors — data-driven priors only reliably assess power for sample sizes smaller or roughly equal to the pilot data
- The study targets variance QTL detection or continuous annotations like pseudo time — the article did not address variance QTLs (no effect size priors) and pseudo time would need discretization

## Inputs

- scPower example priors: eqtl.ref.study ('Blueprint (Monocytes)'), de.ref.study ('Pancreas_alphabeta'), gamma.mixed.fits, disp.fun.param
- Read-UMI fit for the technology, e.g. read.umi.fit '10X_PBMC_1'
- Prior sets of DEGs/eQTL genes with expression ranks and effect sizes (e.g. 84 iCLL vs mCLL DEGs, median |logFC| 2.8)
- Budget specification (e.g. totalBudget 10,000 or 30,000 EUR) and cells-per-individual / read-depth grids (e.g. seq(1000,10000,by=1000), seq(10000,50000,by=5000))
- Optional: pilot scRNA-seq UMI count matrix (e.g. from GSE185714) for fitting expression priors via create.pseudobulk, nbinom.estimation, and mixed.gamma.estimation

## Outputs

- Power grid table (e.g. optimal_design_power_grid.csv) with sampleSize, cells per individual, read depth, cost, and powerDetect
- Optimal design parameters: sample size, cells per individual, and read depth maximizing powerDetect within budget
- Power surfaces visualized with visualize.power.grid (totalCells vs readDepth, filled by sampleSize and powerDetect)

## How to apply

Load the scPower R package with its example priors (eqtl.ref.study 'Blueprint (Monocytes)', de.ref.study e.g. 'Pancreas_alphabeta', gamma.mixed.fits, disp.fun.param, read.umi.fit '10X_PBMC_1'). Model overall detection power as the product of a gene's expression probability (estimated with estimate.exp.prob.param at min.counts=3 and perc.indiv=0.5) and its pseudobulk negative binomial DE or eQTL power, applying multiple testing correction (Bonferroni for eQTL, FDR at sign.threshold=0.05 for DE) using the estimated number of expressed genes. Define a cost model (e.g. 10X droplet: costKit=5600 EUR, costFlowCell=14032 EUR, readsPerFlowcell=4.1e6, samplesPerLane=4, mappingEfficiency=0.8) so library preparation plus sequencing costs sum to the experimental cost. For a restricted budget, sweep grids of cells per individual and read depth with optimize.constant.budget, letting sample size be inferred from the remaining budget, then select the row maximizing powerDetect (e.g. DE 10,000 EUR optimum: 1200 cells, 4 samples, 30,000 reads/cell; eQTL 30,000 EUR optimum: 1500 cells, 242 samples, 10,000 reads/cell). Validate that the chosen combination's computed cost does not exceed the budget and that its powerDetect is the grid maximum; visualize surfaces with visualize.power.grid. If expression cutoffs need tuning, use select.cutoffs, noting best-practice cutoffs remove 19–33% of lowly expressed genes (1–10 reads per million / 1–5 UMI counts).

## Related tools

- **scPower** (R package providing the analytical power model, priors, budget-constrained optimization (optimize.constant.budget), and visualization (visualize.power.grid)) — https://github.com/heiniglab/scPower
- **R** (Language in which scPower is implemented and all power/design calculations are executed) — https://github.com/heiniglab/scPower
- **edgeR** (Pseudobulk DE method whose performance motivates the negative binomial pseudobulk power model used for the DE power term)
- **limma-voom** (Pseudobulk DE method cited alongside edgeR as outperforming single-cell-specific methods, motivating the analytic pseudobulk model)

## Examples

```
library(scPower); opt <- optimize.constant.budget(totalBudget=10000, nCellsRange=seq(1000,10000,by=1000), readDepthRange=seq(10000,50000,by=5000), read.umi.fit=read.umi.fit$"10X_PBMC_1", costKit=5600, costFlowCell=14032)
```

## Evaluation signals

- The selected parameter combination's computed experimental cost (library preparation + sequencing) does not exceed the specified totalBudget
- The chosen design's powerDetect equals the maximum over the evaluated cells-per-individual × read-depth grid
- Grid values reproduce paper-reported optima, e.g. DE (10,000 EUR): 1200 cells, 4 samples, 30,000 reads/cell; eQTL (30,000 EUR): 1500 cells, 242 samples, 10,000 reads/cell (Fig. 4)
- With iCLL vs mCLL fold-change priors (84 DEGs, median |logFC| 2.8), maximum overall detection power reaches 74% at 3000 cells per cell type and 20 samples
- Expression probability estimates use min.counts=3 and perc.indiv=0.5, and multiple testing correction uses Bonferroni (eQTL) or FDR at 0.05 (DE) based on the estimated number of expressed genes

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger sample sizes can identify new significant DEGs with lower effect sizes not captured in the priors
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default assumption
- Power for detection of variance QTLs from scRNA-seq data was not addressed due to lack of data-driven priors for effect sizes
- Continuous cell annotations such as pseudo time would need to be discretized before the power analysis

## Evidence

- [methods] overall power as product of expression and eQTL/DE power: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [methods] budget-constrained optimization: "scPower gives the user the opportunity to select the best parameter combination for a restricted budget."
- [abstract] shallow vs deep sequencing trade-off: "In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells."
- [readme] package availability: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
- [methods] cost model composition: "The experimental costs are the sum of the library preparation cost and the sequencing costs."
- [discussion] cutoff guidance: "best practice guidelines for differential gene expression with RNA-seq recommend cutoffs that remove between 19 and 33% of lowly expressed genes"
