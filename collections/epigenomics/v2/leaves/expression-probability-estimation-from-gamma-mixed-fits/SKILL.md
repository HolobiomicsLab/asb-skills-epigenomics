---
name: expression-probability-estimation-from-gamma-mixed-fits
description: Use when designing or powering an interindividual scRNA-seq DE/eQTL study
  and you need the probability that a gene passes expression detection thresholds
  (e.g. min.counts=3, perc.indiv=0.5) at given cells-per-sample and read depth.
when_to_use_negative:
- When the goal is single-cell-level (not pseudobulk/interindividual) DE testing power
  — the model was specifically developed for the pseudobulk approach
- When a pilot study's priors would be extrapolated far beyond its sample size — data
  driven priors only reliably assess overall power for sample sizes smaller or roughly
  equal to the pilot data sets
- When continuous cell annotations (e.g. pseudotime) define the comparison — they
  would need to be discretized before power analysis
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: scPower
  role: R package providing estimate.exp.prob.param, calculate.gene.counts, mixed.gamma.estimation,
    visualize.gamma.fits and convert.gamma.parameters for expression probability estimation
    from gamma-mixed fits
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower functions are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:d96176b865aae8a029d8d78e1e0a3e8e63cf229de9b08b43993075b13dc482c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Expression probability estimation from gamma-mixed fits

## Summary

Estimates the single-cell detection power of a gene (probability it is expressed/observed at a given sequencing depth and cell number) using scPower's zero-inflated gamma-mixed fits of expression rank/mean distributions, which are combined multiplicatively with pseudobulk DE power to yield overall detection power.

## When to use

Use when designing or powering an interindividual scRNA-seq DE/eQTL study and you need the probability that a gene passes expression detection thresholds (e.g. min.counts=3, perc.indiv=0.5) at given cells-per-sample and read depth.

## When NOT to use

- When the goal is single-cell-level (not pseudobulk/interindividual) DE testing power — the model was specifically developed for the pseudobulk approach
- When a pilot study's priors would be extrapolated far beyond its sample size — data driven priors only reliably assess overall power for sample sizes smaller or roughly equal to the pilot data sets
- When continuous cell annotations (e.g. pseudotime) define the comparison — they would need to be discretized before power analysis

## Inputs

- scPower gamma.mixed.fits expression prior fits (zero component + two left-censored gamma components)
- read.umi.fit object, e.g. '10X_PBMC_1' from the PBMC study
- disp.fun.param dispersion function parameters for the platform
- Design grid: sample size, cells per cell type, and read depth per cell type
- Optional: a pilot UMI count matrix for fitting new expression priors (e.g. GSE185714 toy dataset)
- Gene set with expression ranks and effect sizes (e.g. 84 iCLL vs mCLL DEGs, median |logFC| 2.8)

## Outputs

- Per-gene expression probabilities for each (sample size, cells per cell type, read depth) design point
- Number of expressed genes per design, used for FDR-based multiple testing correction
- Power grid of overall detection power (expression probability × DE power), e.g. written to power_grid_icll_mcll.csv

## How to apply

Load scPower in R and use the fitted expression priors (e.g. read.umi.fit '10X_PBMC_1', gamma.mixed.fits, disp.fun.param). For each (sample size, cells per cell type, read depth) grid point, call estimate.exp.prob.param with a chosen read depth per cell type, min.counts=3 and perc.indiv=0.5 to obtain per-gene expression probabilities, then use calculate.gene.counts to derive the number of expressed genes that feeds the FDR multiple-testing correction. If new priors must be estimated, run meanUMI.calculation on the count matrix, fit a mixed distribution of a zero component and two left-censored gamma components over gene mean values with mixed.gamma.estimation (num.genes.kept, censoredPoint), optionally visualize with visualize.gamma.fits, and convert rate-shape parameters to mean-standard-deviation parameters with convert.gamma.parameters; gene means can be sampled via sample.mean.values.random or sample.mean.values.quantiles. The rationale is that the analytic model quantifies sparsity-driven detectability separately from DE power, so overall detection power is the product of expression probability and DE power.

## Related tools

- **scPower** (R package providing estimate.exp.prob.param, calculate.gene.counts, mixed.gamma.estimation, visualize.gamma.fits and convert.gamma.parameters for expression probability estimation from gamma-mixed fits) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower functions are executed)

## Examples

```
library(scPower); exp.prob <- estimate.exp.prob.param(exp.prior=gamma.mixed.fits, read.umi.fit=read.umi.fit$`10X_PBMC_1`, read.depth=30000, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- Expression probabilities decrease monotonically as min.counts or perc.indiv thresholds are tightened, and increase with read depth
- Number of expressed genes from calculate.gene.counts falls within the 19–33% removal range recommended for RNA-seq DGE cutoffs (1–10 reads per million / 1–5 UMI counts) when typical cutoffs are applied
- Gamma-mixed fit can be inspected with visualize.gamma.fits: a zero component plus two left-censored gamma components should reproduce the gene mean distribution at the chosen censoredPoint
- Combined with DE power, the overall detection power reproduces paper-validated values, e.g. 74% maximum for the iCLL vs mCLL prior set (84 DEGs, median |logFC| 2.8) at 3000 cells per cell type and 20 samples
- convert.gamma.parameters output is consistent (mean/SD parameters derivable from the input rate-shape parameters)

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger studies find new significant DEGs with lower effect sizes not captured by pilot-based priors
- Actual experiments may have higher doublet rates and fewer usable cells than assumed
- Expression priors must come from well-matched pilot experiments or literature; mismatched platforms/tissues bias probabilities
- Continuous cell annotations such as pseudotime must be discretized before the analysis

## Evidence

- [methods] gamma-mixed fit workflow: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] parameter conversion: "converted from rate-shape parameters to mean-standard deviation parameters by "convert.gamma.parameters""
- [methods] sampling strategy: "either random (sample.mean.values.random) or using the quantiles of the gamma mixed distribution (sample.mean.values.quantiles)"
- [methods] overall power model: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [discussion] cutoff guidance: "best practice guidelines for differential gene expression with RNA-seq recommend cutoffs that remove between 19 and 33% of lowly expressed genes"
- [readme] package scope: "Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets."
