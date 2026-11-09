---
name: umi-dependent-gamma-parameter-regression
description: Use when constructing a cell type-specific expression prior for scPower
  power analysis from pilot scRNA-seq UMI count matrices, requiring gene-mean distributions
  parameterized by sequencing depth (mean UMI per cell).
when_to_use_negative:
- The task is downstream power calculation for DEG/eQTL studies using an already-fitted
  expression prior (e.g. package-supplied 10X_PBMC_1) rather than fitting new gamma
  regressions from pilot data
- The data are not per-gene mean expression distributions from single cell counts
  — e.g. bulk RNA-seq or pre-aggregated expression summaries with no per-cell UMI
  counts to anchor the mean-UMI regression
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package providing mixed.gamma.estimation, meanUMI.calculation, convert.gamma.parameters,
    umi.gamma.relation, umi.read.relation and dispersion.function.estimation used
    to build the expression prior
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower prior-fitting workflow is executed
    and the prior serialized as expression_prior.RData
  repo: https://github.com/heiniglab/scPower
- name: DESeq
  role: Size-factor normalization (standard, or poscounts fallback) applied inside
    nbinom.estimation before gene means are fitted with the gamma mixture
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:1b596136bd8d8a437eeafa0618b51e9b69ba94843251459b611931597e008c15
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# umi-dependent-gamma-parameter-regression

## Summary

Fits a zero-inflated gamma mixed distribution to per-gene normalized mean expression in a cell type, then regresses the gamma parameters (mean–sd parameterization) on mean UMI counts to build the scPower expression prior that predicts gene detectability at any sequencing depth.

## When to use

Use when constructing a cell type-specific expression prior for scPower power analysis from pilot scRNA-seq UMI count matrices, requiring gene-mean distributions parameterized by sequencing depth (mean UMI per cell).

## When NOT to use

- The task is downstream power calculation for DEG/eQTL studies using an already-fitted expression prior (e.g. package-supplied 10X_PBMC_1) rather than fitting new gamma regressions from pilot data
- The data are not per-gene mean expression distributions from single cell counts — e.g. bulk RNA-seq or pre-aggregated expression summaries with no per-cell UMI counts to anchor the mean-UMI regression

## Inputs

- 2D UMI count matrices from a pilot droplet-based study at multiple simulated read depths (e.g. 100%, 75%, 50%, 25% subsampled, such as the scPower toy data from GSE185714)
- Cell annotation data frame with cell type and donor columns whose rows match count-matrix column ordering
- Cellranger mapped read counts per sequencing depth for fitting the read-depth-to-UMI relationship

## Outputs

- gamma.linear.fit.new: linear regression of gamma mean and sd parameters on mean UMI, annotated with the cell type (ct column)
- disp.fun.general.new: median mean-dispersion function over subsampling runs per cell type
- read.umi.fit.new: logarithmic read-depth-to-UMI fit
- expression_prior.RData serializing these objects for downstream estimate.exp.prob.param calls

## How to apply

First create a 3D pseudobulk matrix with create.pseudobulk and extract expressed genes with calculate.gene.counts (min.counts=3, perc.indiv=0.5), then estimate per-gene negative binomial means with nbinom.estimation (DESeq normalization, sizeFactorMethod='standard', falling back to 'poscounts' for very sparse data). Fit the gene-mean distribution with mixed.gamma.estimation per depth (num.genes.kept=21000, censoredPoint=1/num_cells_celltype), using a zero component plus two left-censored gamma components; optionally verify fits by simulating means with sample.mean.values.random/quantiles or visualize.gamma.fits. Compute mean UMI per cell with meanUMI.calculation, convert gamma shape-rate parameters to mean–sd form with convert.gamma.parameters, and regress the gamma parameters linearly on mean UMI with umi.gamma.relation so priors transfer across read depths. Take the median mean-dispersion parameters over subsampling runs with dispersion.function.estimation, and fit the log read-depth-to-UMI relation with umi.read.relation (e.g. cellranger mapped reads 23130, 17422, 11666, 5859). The rationale: expression probability in a planned experiment depends on the mean expression a gene would have at a given depth, so encoding gamma parameters as smooth functions of mean UMI lets estimate.exp.prob.param predict expressed gene counts at any design point.

## Related tools

- **scPower** (R package providing mixed.gamma.estimation, meanUMI.calculation, convert.gamma.parameters, umi.gamma.relation, umi.read.relation and dispersion.function.estimation used to build the expression prior) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower prior-fitting workflow is executed and the prior serialized as expression_prior.RData) — https://github.com/heiniglab/scPower
- **DESeq** (Size-factor normalization (standard, or poscounts fallback) applied inside nbinom.estimation before gene means are fitted with the gamma mixture)

## Examples

```
gamma.fit.temp <- mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)
```

## Evaluation signals

- Validate the fitted prior by predicting expressed gene numbers with estimate.exp.prob.param (min.counts=3, perc.indiv=0.5) and comparing against the observed gene counts in expressed.genes.df
- Check gamma fits visually or by simulation using visualize.gamma.fits, sample.mean.values.random and sample.mean.values.quantiles — simulated means/quantiles should reproduce the empirical gene-mean distribution
- Annotation sanity: every gamma and dispersion fit row carries the correct cell type in the ct column, and annotation rows match count-matrix column ordering before pseudobulking
- Gamma parameters are in mean–sd (not shape–rate) form after convert.gamma.parameters, and the linear fits of mean and sd against mean UMI show monotone, depth-consistent trends across all four subsampled matrices
- read.umi.fit.new reproduces the observed UMI counts at the mapped-read depths (23130, 17422, 11666, 5859) within expected error

## Limitations

- Data-driven priors only reliably assess overall power for sample sizes smaller than or roughly equal to the pilot data set sample size
- The gamma fit is left-censored at censoredPoint=1/num_cells_celltype with num.genes.kept=21000; very sparse cell types or very low-depth matrices may need the poscounts size-factor fallback in nbinom.estimation
- Actual experiments may show higher doublet rates and fewer usable cells than assumed when translating the prior into design predictions

## Evidence

- [methods] A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] gamma.fit.temp call with num.genes.kept and censoredPoint: "gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)"
- [methods] mean UMI calculation: "calculated from the original count matrices using the function "meanUMI.calculation""
- [methods] gamma parameter conversion to mean-sd form: "converted from rate-shape parameters to mean-standard deviation parameters by "convert.gamma.parameters""
- [readme] priors can be estimated from new data sets: "Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets."
