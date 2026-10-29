---
name: sample-size-and-cells-per-sample-design-grid-evaluation
description: Use when planning an interindividual single-cell DE (or eQTL) study and
  you need to pick sample size, cells per cell type, and read depth by comparing power
  across a design grid rather than guessing a single design.
when_to_use_negative:
- The planned sample size far exceeds the pilot/reference study the priors were derived
  from — data-driven priors only reliably assess power at sample sizes smaller or
  roughly equal to the pilot data sets
- The study is a simulation-based power check of a single fixed design with complex
  cell-type annotation needs, where tools like muscat or powsimR are more appropriate
  (despite their 35–48 GB memory footprint)
- The comparison design is unbalanced or non-two-group (e.g. continuous covariates
  or pseudo-time), which the balanced pseudobulk NB power model here does not cover
  directly
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package providing the analytic pseudobulk power model, expression probability
    estimation (estimate.exp.prob.param), and grid optimization of sample size, cells
    per sample, and read depth
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Environment in which scPower is installed and the design grid is evaluated
    and written to CSV
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:908d8548c0765ea1eadc216983c50ac6c8f5b744f5c2ac45d865087c1f7563fa
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sample-size-and-cells-per-sample-design-grid-evaluation

## Summary

Grid-based evaluation of sample size and cells-per-sample combinations using scPower's analytic pseudobulk power model, computing overall detection power as expression probability × DE power for each design point to identify the optimal experimental parameter combination.

## When to use

Use when planning an interindividual single-cell DE (or eQTL) study and you need to pick sample size, cells per cell type, and read depth by comparing power across a design grid rather than guessing a single design.

## When NOT to use

- The planned sample size far exceeds the pilot/reference study the priors were derived from — data-driven priors only reliably assess power at sample sizes smaller or roughly equal to the pilot data sets
- The study is a simulation-based power check of a single fixed design with complex cell-type annotation needs, where tools like muscat or powsimR are more appropriate (despite their 35–48 GB memory footprint)
- The comparison design is unbalanced or non-two-group (e.g. continuous covariates or pseudo-time), which the balanced pseudobulk NB power model here does not cover directly

## Inputs

- Effect size and expression-rank priors from a DE reference study (e.g. scPower::de.ref.study, 84 iCLL vs mCLL DEGs, median |logFC| 2.8)
- Package example fits: read.umi.fit ('10X_PBMC_1'), gamma.mixed.fits, disp.fun.param
- Design grid of sample sizes, cells per cell type, and read depth per cell type

## Outputs

- Power grid table (e.g. power_grid_icll_mcll.csv) with overall detection power per design point
- Optimal (sample size, cells per cell type, read depth) combination and its maximum overall detection power (74% in the reference case)

## How to apply

Assemble matched priors (e.g. the iCLL vs mCLL DE reference set of 84 DEGs with median |logFC| 2.8, expression ranks, plus package fits read.umi.fit '10X_PBMC_1', gamma.mixed.fits, and disp.fun.param). For each (sample size, cells per cell type, read depth) combination in the grid, estimate per-gene expression probabilities with scPower's estimate.exp.prob.param using min.counts=3 and perc.indiv=0.5, then derive the number of expressed genes. Compute DE power per gene with the pseudobulk negative binomial power model for a balanced two-group comparison, scaling pseudobulk means by cells per sample and dispersions inversely. Apply FDR multiple-testing correction at sign.threshold=0.05 based on the estimated number of expressed genes, form overall detection power = expression probability × DE power, and average over the prior gene set to fill the power grid. In the reference case (iCLL vs mCLL priors), the grid's maximum of 74% overall detection power was reached at 3000 cells per cell type with 20 samples.

## Related tools

- **scPower** (R package providing the analytic pseudobulk power model, expression probability estimation (estimate.exp.prob.param), and grid optimization of sample size, cells per sample, and read depth) — https://github.com/heiniglab/scPower
- **R** (Environment in which scPower is installed and the design grid is evaluated and written to CSV) — https://github.com/heiniglab/scPower

## Examples

```
scPower::estimate.exp.prob.param(read.umi.fit=read.umi.fit[["10X_PBMC_1"]], gamma.mixed.fits=gamma.mixed.fits, disp.fun.param=disp.fun.param, sample.size=20, nCells=3000, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- The power grid reproduces the reference maximum of 74% overall detection power at 3000 cells per cell type and sample size 20 for the iCLL vs mCLL 84-DEG prior set
- Per-gene overall detection power equals expression probability × DE power for every design point in the grid
- The number of expressed genes used for FDR correction matches counts derived with min.counts=3 and perc.indiv=0.5
- The grid shows the paper's qualitative trend that shallow sequencing of many cells outperforms deep sequencing of fewer cells
- Output table (power_grid_icll_mcll.csv) contains one row per (sample size, cells per cell type, read depth) combination with finite power values

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger sample sizes would identify new significant DEGs with lower effect sizes not captured in the priors
- Actual experiments might show higher doublet rates and fewer usable cells than the default assumptions
- Simulation-based alternatives (muscat, powsimR) need 35–48 GB memory at sample size 20 with 3000 cells, but scPower's analytic model trades simulation fidelity for speed

## Evidence

- [methods] overall detection power as product: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [abstract] shallow vs deep sequencing: "In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells."
- [methods] best parameter combination for budget: "scPower gives the user the opportunity to select the best parameter combination for a restricted budget."
- [discussion] pilot sample size limitation: "our data driven priors only allow for reliably assessing the overall power in sample sizes that are smaller or roughly equal to the sample size of the pilot data sets"
- [readme] README power calculation purpose: "It enables the user to calculate the power for a given experimental setup and to choose for a restricted budget the optimal combination of experimental parameters which maximizes the power."
