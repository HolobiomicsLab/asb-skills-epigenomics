---
name: pseudobulk-negative-binomial-power-modeling
description: 'Use when planning or optimizing multi-sample scRNA-seq DEG studies:
  you need power/sample-size/read-depth trade-offs (e.g. cells per cell type vs number
  of individuals) for a pseudobulk two-group comparison with effect-size and expression
  priors.'
when_to_use_negative:
- The analysis goal is single-cell-level DE methods or mixed models rather than the
  pseudobulk approach (scPower's model was specifically developed for pseudobulk DE
  analysis)
- The study design is not a (balanced) interindividual two-group comparison, e.g.
  pseudotime or other continuous annotations, which would first need discretization
  and are outside the model
- Sample sizes far exceed the pilot/reference study used to derive the priors — data-driven
  priors reliably assess power only for sample sizes smaller or roughly equal to the
  pilot data set
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: 'R package implementing the analytic power model: expression probability estimation
    (estimate.exp.prob.param), pseudobulk negative binomial DE power, FDR-based multiple
    testing correction, and design optimization'
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment; scPower is an open source R package with Shiny app and
    vignettes (introduction-scPower, reproduce-paper-plots)
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:4a9130f9cb42e184454f9e590108e889e0778d5dd9da2720c875b834a2f29611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pseudobulk-negative-binomial-power-modeling

## Summary

Analytic power estimation for interindividual differential expression in single-cell RNA-seq using a pseudobulk negative binomial model, combining per-gene expression probability with DE power (overall detection power = expression probability × DE power), as implemented in the scPower R package.

## When to use

Use when planning or optimizing multi-sample scRNA-seq DEG studies: you need power/sample-size/read-depth trade-offs (e.g. cells per cell type vs number of individuals) for a pseudobulk two-group comparison with effect-size and expression priors.

## When NOT to use

- The analysis goal is single-cell-level DE methods or mixed models rather than the pseudobulk approach (scPower's model was specifically developed for pseudobulk DE analysis)
- The study design is not a (balanced) interindividual two-group comparison, e.g. pseudotime or other continuous annotations, which would first need discretization and are outside the model
- Sample sizes far exceed the pilot/reference study used to derive the priors — data-driven priors reliably assess power only for sample sizes smaller or roughly equal to the pilot data set

## Inputs

- Effect-size priors for a DEG set (gene list with log fold changes, e.g. iCLL vs mCLL 84 DEGs, median |logFC| 2.8) from a reference DE study
- Expression priors: per-cell-type gamma mixed distribution fits over gene mean values
- read.umi.fit / UMI count distribution fit for the scRNA-seq platform (e.g. 10X_PBMC_1)
- Dispersion mean-dispersion function (disp.fun.param)
- Experimental design grid: sample size, cells per cell type, sequencing depth/read depth per cell

## Outputs

- Per-gene and average overall detection power (expression probability × DE power) for each design in the parameter grid
- Estimated number of expressed genes per design (from min.counts=3, perc.indiv=0.5 thresholds)
- Power grid table (e.g. power_grid_icll_mcll.csv) and optimal experimental parameter combination

## How to apply

Assemble effect-size priors from a reference DE study (e.g. 84 iCLL vs mCLL DEGs with median |logFC| 2.8) together with expression distribution priors (gamma mixed fits, read UMI fits such as '10X_PBMC_1', and a dispersion function). For each (sample size, cells per cell type, read depth) grid point, estimate per-gene expression probabilities with estimate.exp.prob.param using min.counts=3 and perc.indiv=0.5 to derive the number of expressed genes. Compute DE power per gene with the pseudobulk negative binomial model for a balanced two-group comparison, scaling pseudobulk means by the number of cells per sample and dispersions by their inverse. Correct for multiple testing with MTmethod='FDR' at sign.threshold=0.05 based on the estimated number of expressed genes. Multiply expression probability by DE power per gene and average over the prior gene set to fill the power grid; e.g. the iCLL vs mCLL prior set reaches a maximum overall detection power of 74% at 3000 cells per cell type with 20 samples (10 individuals per group).

## Related tools

- **scPower** (R package implementing the analytic power model: expression probability estimation (estimate.exp.prob.param), pseudobulk negative binomial DE power, FDR-based multiple testing correction, and design optimization) — https://github.com/heiniglab/scPower
- **R** (Runtime environment; scPower is an open source R package with Shiny app and vignettes (introduction-scPower, reproduce-paper-plots)) — https://github.com/heiniglab/scPower

## Examples

```
exp.prob <- estimate.exp.prob.param(read.umi.fit=scPower::read.umi.fit$"10X_PBMC_1", gamma.mixed.fit=scPower::gamma.mixed.fits, disp.fun.param=scPower::disp.fun.param, sampleSize=20, cellsPerCelltype=3000, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- Reproducing the paper's benchmark: the iCLL vs mCLL prior set (84 DEGs, median |logFC| 2.8) must yield a maximum overall detection power of 74% at 3000 cells per cell type and a total balanced sample size of 20
- Per-design DE power must be FDR-corrected with MTmethod='FDR' at sign.threshold=0.05 using the number of expressed genes estimated under min.counts=3 and perc.indiv=0.5
- Pseudobulk means must scale linearly with cells per sample and dispersions inversely with cells per sample before power computation
- Every power-grid cell must equal the average over the prior gene set of expression probability × DE power (product model, not a sum)
- Optimal designs should reflect the paper's qualitative finding that shallow sequencing of high numbers of cells outperforms deep sequencing of fewer cells

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger sample sizes identify new significant DEGs with lower effect sizes not captured in the power computation
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default assumptions
- A sample size of 20 with 3000 cells per sample is computationally heavy for simulation-based alternatives (35 GB for muscat, 48 GB for powsimR), which is why the analytic model is preferred for large design grids

## Evidence

- [methods] We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene.: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [intro] the pseudobulk approach in combination with classical differential gene expression methods such as edgeR: "the “pseudobulk” approach in combination with classical differential gene expression methods such as edgeR44 and limma-voom45 outperforms single cell specific methods"
- [discussion] best practice guidelines for differential gene expression with RNA-seq recommend cutoffs: "best practice guidelines for differential gene expression with RNA-seq recommend cutoffs that remove between 19 and 33% of lowly expressed genes"
- [readme] scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq.: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
- [abstract] In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells.: "In general, shallow sequencing of high numbers of cells leads to higher overall power than deep sequencing of fewer cells."
