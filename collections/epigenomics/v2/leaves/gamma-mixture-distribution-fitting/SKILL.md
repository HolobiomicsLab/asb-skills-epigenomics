---
name: gamma-mixture-distribution-fitting
description: Use when building expression probability priors from scRNA-seq count
  matrices for scPower power analysis, i.e. when per-gene negative-binomial means
  must be summarized into a fitted distribution of expression means per read depth.
when_to_use_negative:
- Input is already a fitted prior from the package (e.g. gamma.mixed.fits) and no
  new dataset-specific expression distribution needs to be estimated
- The goal is simulation-based power analysis of individual count matrices (e.g. with
  muscat or powsimR) rather than analytic priors for pseudobulk power estimation
- Cell-type annotations are unavailable or do not match the count matrix column ordering,
  so no reliable pseudobulk mean values can be formed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation,
    visualize.gamma.fits, and convert.gamma.parameters for fitting expression priors
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower fitting functions are executed
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:a9d172f0fdf5bcc0fa473210cc156db18489505e83c7dc1e5ec4f43958bd1167
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# gamma-mixture-distribution-fitting

## Summary

Fit a gamma mixed distribution (a zero component plus two left-censored gamma components) to per-gene pseudobulk mean values in scPower to obtain expression-distribution priors used for analytic power estimation in single-cell DE/eQTL study design.

## When to use

Use when building expression probability priors from scRNA-seq count matrices for scPower power analysis, i.e. when per-gene negative-binomial means must be summarized into a fitted distribution of expression means per read depth.

## When NOT to use

- Input is already a fitted prior from the package (e.g. gamma.mixed.fits) and no new dataset-specific expression distribution needs to be estimated
- The goal is simulation-based power analysis of individual count matrices (e.g. with muscat or powsimR) rather than analytic priors for pseudobulk power estimation
- Cell-type annotations are unavailable or do not match the count matrix column ordering, so no reliable pseudobulk mean values can be formed

## Inputs

- 2D scRNA-seq UMI/read count matrix with donor and cell-type annotation data frames (e.g. droplet lung atlas, Smart-seq2 pancreas atlas)
- Read-depth subsampled pseudobulk mean values per gene from nbinom.estimation
- Number of cells per cell type (to set the censoredPoint)
- Package example priors such as gamma.mixed.fits and disp.fun.param

## Outputs

- Fitted gamma mixed distribution parameters (zero component plus two left-censored gamma components) per read depth
- Mean-sd parameterized gamma parameters (after convert.gamma.parameters) for downstream expression probability prediction
- Predicted expressed gene counts vs. observed counts for r2 validation

## How to apply

First reformat the 2D count matrix into a 3D pseudobulk matrix with create.pseudobulk using matching cell-type annotations, then estimate per-gene negative binomial mean and dispersion with nbinom.estimation (DESeq-based normalization) for each read-depth subsample. Next fit the gamma mixed distribution over the mean value vectors with mixed.gamma.estimation, using num.genes.kept=21000 and a left-censoring point of censoredPoint=1/num_cells_per_celltype; visualize the quality of the fit with visualize.gamma.fits. For droplet/UMI technologies, subsequently convert gamma rate-shape parameters to mean-standard-deviation parameterization with convert.gamma.parameters and relate them to mean UMI per cell via meanUMI.calculation and the read-UMI fit; for read-count technologies like Smart-seq2, parameterize the gamma-mixed distribution directly over read depth with technology-specific settings (e.g. gene-length-normalized min.norm.count=3 and a read-depth-dependent dispersion function). The rationale is that these data-driven priors quantify the probability that a gene is expressed in a cell type, which scPower multiplies with eQTL/DE power to obtain the overall detection power.

## Related tools

- **scPower** (R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation, visualize.gamma.fits, and convert.gamma.parameters for fitting expression priors) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower fitting functions are executed)

## Examples

```
gamma.fit.temp <- mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)
```

## Evaluation signals

- r2 between predicted and observed expressed gene numbers reaches ~0.995 for Drop-seq lung and ~0.991 for Smart-seq2 pancreas (Supplementary Table S3)
- Fitted distribution uses num.genes.kept = 21000 and censoredPoint = 1/num_cells_per_celltype
- visualize.gamma.fits plots show the zero component plus two gamma components match the empirical mean value distribution
- Predictions are generated with min.counts=3 and perc.indiv=0.5 and validated against counts from calculate.gene.counts

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Actual experiments might result in higher doublet rates and lower number of usable cells than the default assumption
- Continuous cell annotations such as pseudo time would need to be discretized before the power analysis

## Evidence

- [methods] A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] mixed.gamma.estimation call with parameters: "gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)"
- [methods] negative binomial estimation precedes the gamma fit: "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
- [readme] priors can be estimated from new data sets: "Necessary experimental priors, e.g. effect sizes and expression distributions, can be taken from example data sets, saved in the package, or estimated from new data sets."
