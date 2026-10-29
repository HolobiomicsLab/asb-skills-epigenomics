---
name: zero-inflated-gamma-mixture-fitting
description: 'Use when building scPower expression priors from a pilot UMI count matrix:
  you have pseudobulk-normalized gene means per cell type and need a zero-inflated
  gamma model over the means to later compute expression probabilities at chosen read
  depths.'
when_to_use_negative:
- The task is differential expression or power calculation with existing published
  priors (e.g. 10X_PBMC_1 or Pancreas_alphabeta reference studies) rather than fitting
  new priors from a pilot dataset
- Data are full-length read-count based with radically different count characteristics
  and no UMI matrix available for pseudobulk mean estimation
- The planned study sample size far exceeds the pilot dataset, since data-driven priors
  only reliably assess overall power for sample sizes smaller or roughly equal to
  the pilot data
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPower
  role: R package providing mixed.gamma.estimation, convert.gamma.parameters, umi.gamma.relation,
    dispersion.function.estimation, umi.read.relation and estimate.exp.prob.param
    for fitting and applying the expression prior
  repo: https://github.com/heiniglab/scPower
- name: DESeq
  role: Normalization used inside nbinom.estimation to obtain per-gene pseudobulk
    means prior to gamma mixture fitting
- name: R
  role: Language in which the fitting pipeline and prior serialization are executed
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:7703215c5ff36737307b6dda1df2c5b6bf89a42f13204a01c499444df2df1f8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - DESeq
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# zero-inflated gamma mixture fitting

## Summary

Fits a mixed distribution of a zero component plus two left-censored gamma components to per-gene pseudobulk mean expression values, then re-parameterizes and serializes the gamma fits (with dispersion and read-depth curves) as an scPower expression prior for predicting cell type-specific expression probability in planned scRNA-seq experiments.

## When to use

Use when building scPower expression priors from a pilot UMI count matrix: you have pseudobulk-normalized gene means per cell type and need a zero-inflated gamma model over the means to later compute expression probabilities at chosen read depths.

## When NOT to use

- The task is differential expression or power calculation with existing published priors (e.g. 10X_PBMC_1 or Pancreas_alphabeta reference studies) rather than fitting new priors from a pilot dataset
- Data are full-length read-count based with radically different count characteristics and no UMI matrix available for pseudobulk mean estimation
- The planned study sample size far exceeds the pilot dataset, since data-driven priors only reliably assess overall power for sample sizes smaller or roughly equal to the pilot data

## Inputs

- UMI count matrix (e.g. scPower toy matrix sampled from GSE185714, optionally read-depth-subsampled at 75%, 50%, 25%)
- Cell annotation data frame with cell type and donor columns matching count-matrix column order
- Mapped read counts per sequencing depth (e.g. cellranger: 23130, 17422, 11666, 5859) for umi.read.relation

## Outputs

- gamma.linear.fit.new: linearly regressed gamma mean-sd parameters as function of mean UMI counts
- disp.fun.general.new: mean-dispersion function parameters (median over subsampling runs), annotated with cell type
- read.umi.fit.new: logarithmic read-depth-to-UMI relationship fit
- expression_prior.RData serializing the three fits for estimate.exp.prob.param

## How to apply

Reformat the 2D UMI count matrix into a 3D pseudobulk matrix with create.pseudobulk, estimate per-gene negative binomial means with nbinom.estimation (DESeq normalization, sizeFactorMethod='standard', falling back to 'poscounts' for very sparse data), then run mixed.gamma.estimation on the mean vector with num.genes.kept=21000 and censoredPoint=1/num_cells_celltype. Compute mean UMI per cell via meanUMI.calculation, convert the gamma fits from shape-rate to mean-sd parameterization with convert.gamma.parameters, regress the gamma parameters linearly on mean UMI with umi.gamma.relation, and take the median of mean-dispersion parameters over read-depth subsampling runs with dispersion.function.estimation. Fit the logarithmic read-depth-to-UMI relationship with umi.read.relation (e.g. cellranger mapped-read counts 23130, 17422, 11666, 5859), annotate the fits with the cell type via the ct column, and serialize gamma.linear.fit, disp.fun.general and read.umi.fit as expression_prior.RData. Validate by predicting expressed gene numbers with estimate.exp.prob.param (min.counts=3, perc.indiv=0.5) against observed expressed gene counts; optionally inspect fits by simulating means with sample.mean.values.random or quantiles via visualize.gamma.fits.

## Related tools

- **scPower** (R package providing mixed.gamma.estimation, convert.gamma.parameters, umi.gamma.relation, dispersion.function.estimation, umi.read.relation and estimate.exp.prob.param for fitting and applying the expression prior) — https://github.com/heiniglab/scPower
- **DESeq** (Normalization used inside nbinom.estimation to obtain per-gene pseudobulk means prior to gamma mixture fitting)
- **R** (Language in which the fitting pipeline and prior serialization are executed) — https://github.com/heiniglab/scPower

## Examples

```
gamma.fit <- mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = 1/num_cells_celltype)
```

## Evaluation signals

- estimate.exp.prob.param-predicted expressed gene counts (min.counts=3, perc.indiv=0.5) match the observed counts in expressed.genes.df from calculate.gene.counts
- mixed.gamma.estimation runs with num.genes.kept=21000 and censoredPoint=1/num_cells_celltype, producing a zero component plus two left-censored gamma components per cell type
- Gamma fits are converted from shape-rate to mean-sd parameters and the regressed parameters on mean UMI (gamma.linear.fit.new) show a good linear relationship over the four read-depth matrices
- disp.fun.general.new dispersion parameters equal the median over subsampling runs and are annotated with the correct cell type via the ct column
- Optionally, simulated means from sample.mean.values.random or quantiles reproduce the empirical mean distribution as checked by visualize.gamma.fits

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Fitting priors requires per-cell-type pseudobulk matrices; for very sparse data nbinom.estimation must fall back from sizeFactorMethod='standard' to 'poscounts'
- The default multiplet-rate and usable-cell assumptions may not match actual experiments, which might result in higher doublet rates and lower number of usable cells

## Evidence

- [methods] A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint): "gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)"
- [methods] converted from rate-shape parameters to mean-standard deviation parameters by "convert.gamma.parameters": "converted from rate-shape parameters to mean-standard deviation parameters by "convert.gamma.parameters""
- [methods] A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714): "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714)"
- [readme] Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets.: "Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets."
