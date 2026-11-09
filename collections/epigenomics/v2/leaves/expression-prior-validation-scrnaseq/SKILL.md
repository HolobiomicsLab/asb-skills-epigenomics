---
name: expression-prior-validation-scrnaseq
description: Use when a fitted expression prior (gamma.linear.fit.new, disp.fun.general.new,
  read.umi.fit.new) has just been estimated from pilot scRNA-seq UMI matrices and
  must be checked before use in experimental design or power calculations.
when_to_use_negative:
- The planned experiment uses read-count (full-length) protocols or cell types with
  no matching pilot data — data-driven priors only reliably assess sample sizes smaller
  or roughly equal to the pilot study
- The input is already a published expression prior (e.g. 10X_PBMC_1, Pancreas_alphabeta)
  rather than a newly fitted prior that needs validation
- The goal is DEG/eQTL power itself rather than validating the expression-probability
  component of the model
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPower
  role: Provides estimate.exp.prob.param, calculate.gene.counts, visualize.gamma.fits
    and sample.mean.values.random used to predict and validate expressed gene numbers
    against pilot data
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which the scPower prior-fitting and validation workflow is implemented
    and executed
  repo: https://github.com/heiniglab/scPower
- name: DESeq
  role: Upstream normalization/size-factor estimation (standard, falling back to poscounts)
    feeding the negative binomial means that the prior is fitted on
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:c7a208ce788224d32f2c1027ef45454ea51bd50795d68ddbdd5c82f3f0c1edd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - DESeq
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Expression prior validation for scRNA-seq power analysis

## Summary

Validates fitted scPower expression priors (gamma mixture fits, mean-dispersion curves, read-depth-UMI curves) by predicting expressed gene numbers for a cell type and comparing them against observed expressed gene counts from pilot data, ensuring the prior is trustworthy before it is used for power analysis.

## When to use

Use when a fitted expression prior (gamma.linear.fit.new, disp.fun.general.new, read.umi.fit.new) has just been estimated from pilot scRNA-seq UMI matrices and must be checked before use in experimental design or power calculations.

## When NOT to use

- The planned experiment uses read-count (full-length) protocols or cell types with no matching pilot data — data-driven priors only reliably assess sample sizes smaller or roughly equal to the pilot study
- The input is already a published expression prior (e.g. 10X_PBMC_1, Pancreas_alphabeta) rather than a newly fitted prior that needs validation
- The goal is DEG/eQTL power itself rather than validating the expression-probability component of the model

## Inputs

- Fitted gamma linear fits per cell type (gamma.linear.fit.new, mean-sd parameterized and regressed on mean UMI)
- Fitted mean-dispersion function parameters (disp.fun.general.new, median over subsampling runs)
- Fitted read-depth-to-UMI log relation (read.umi.fit.new) built from cellranger mapped-read counts (23130, 17422, 11666, 5859)
- Observed expressed gene counts per cell type, depth and individual (expressed.genes.df from calculate.gene.counts with min.counts=3, perc.indiv=0.5)
- Pilot UMI count matrices at full and subsampled read depth (e.g. from GSE185714) with cell type / donor annotation

## Outputs

- Predicted number of expressed genes per cell type and sequencing depth (from estimate.exp.prob.param)
- Validation comparison of predicted vs. observed expressed gene counts across all four read-depth levels
- Serialized and validated expression_prior.RData object

## How to apply

Serialize the fitted prior objects (gamma.linear.fit.new, disp.fun.general.new, read.umi.fit.new) as expression_prior.RData, annotating gamma and dispersion fits with the cell type via the ct column. Then call estimate.exp.prob.param with the same detection thresholds used during prior estimation (min.counts=3, perc.indiv=0.5) to predict the number of expressed genes for each sequencing depth and cell type. Compare these predictions against the observed expressed gene counts held in expressed.genes.df (derived from calculate.gene.counts on the pilot pseudobulk matrices); agreement across all read-depth subsampling levels (e.g. 23130, 17422, 11666, 5859 mapped reads) validates the prior. Optionally, gamma fit quality can be inspected beforehand by simulating means with sample.mean.values.random or quantiles via visualize.gamma.fits. The rationale is that the prior is only reliable if the analytic expression-probability model reproduces detectable-gene counts from the pilot data at multiple depths.

## Related tools

- **scPower** (Provides estimate.exp.prob.param, calculate.gene.counts, visualize.gamma.fits and sample.mean.values.random used to predict and validate expressed gene numbers against pilot data) — https://github.com/heiniglab/scPower
- **R** (Language in which the scPower prior-fitting and validation workflow is implemented and executed) — https://github.com/heiniglab/scPower
- **DESeq** (Upstream normalization/size-factor estimation (standard, falling back to poscounts) feeding the negative binomial means that the prior is fitted on)

## Examples

```
predicted <- estimate.exp.prob.param(exp.prior=expression_prior, n.cells=3000, seq.depth=23130, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- Predicted expressed gene numbers from estimate.exp.prob.param (min.counts=3, perc.indiv=0.5) closely match observed counts in expressed.genes.df at all four sequencing depths
- Validation succeeds for every read-depth subsample (23130, 17422, 11666, 5859 mapped reads), not only the full-depth matrix
- Gamma fits annotated with the correct cell type via the ct column; gamma parameters are in mean-sd form after convert.gamma.parameters
- Simulated means from sample.mean.values.random / quantiles reproduce the observed mean distribution when visualized with visualize.gamma.fits
- expression_prior.RData contains all three objects (gamma.linear.fit.new, disp.fun.general.new, read.umi.fit.new) and reloads cleanly for downstream power analysis

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data set
- Larger sample sizes identify new significant DEGs with lower effect sizes not captured in pilot-study-based power computation
- Actual experiments might result in higher doublet rates and lower number of usable cells than assumed

## Evidence

- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
- [methods] gamma mixed distribution fit: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] gamma parameter conversion: "converted from rate-shape parameters to mean-standard deviation parameters by "convert.gamma.parameters""
- [methods] UMI mean calculation: "calculated from the original count matrices using the function "meanUMI.calculation""
- [methods] toy dataset source: "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714)"
- [discussion] pilot prior scope limitation: "our data driven priors only allow for reliably assessing the overall power in sample sizes that are smaller or roughly equal to the sample size of the pilot data sets"
- [readme] package scope from README: "Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets."
