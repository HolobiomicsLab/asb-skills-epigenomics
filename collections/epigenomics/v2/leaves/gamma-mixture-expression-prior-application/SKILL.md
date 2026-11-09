---
name: gamma-mixture-expression-prior-application
description: Use when you have a UMI count matrix with cell-type annotations and need
  to predict (or validate) the number of detectable genes per cell type as a function
  of cells per cell type and transcriptome-mapped read depth.
when_to_use_negative:
- The goal is power for differential expression or eQTL detection rather than expression
  detection probability — use scPower's DE/eQTL power functions instead
- The count data are read-based or from a technology not matched by the fitted read-UMI
  and gamma priors; refit priors (nbinom.estimation, mixed.gamma.estimation) on matched
  pilot data first
- Cell annotations are continuous (e.g. pseudotime) and not yet discretized into cell
  types
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPower
  role: R package providing create.pseudobulk, calculate.gene.counts, estimate.exp.prob.param
    and the gamma-mixture prior fitting used to predict expressed gene counts
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment; scPower is an open source R package (installed via devtools::install_github)
  repo: https://github.com/heiniglab/scPower
- name: ggplot2
  role: Plotting predicted vs observed expressed gene counts against transcriptome.mapped.reads
    (with reshape2::melt)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:2ece7f6de84df19c27bca9b8d5371c5149adfca9441056144c9a3208ea442137
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - ggplot2
  - scPower
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gamma-mixture-expression-prior-application

## Summary

Apply scPower's fitted gamma mixed-distribution expression priors (read-UMI fit and gamma.linear.fit) to predict the number of expressed genes per cell type for a planned scRNA-seq experiment, and validate the predictions against pseudobulk-observed expressed gene counts.

## When to use

Use when you have a UMI count matrix with cell-type annotations and need to predict (or validate) the number of detectable genes per cell type as a function of cells per cell type and transcriptome-mapped read depth.

## When NOT to use

- The goal is power for differential expression or eQTL detection rather than expression detection probability — use scPower's DE/eQTL power functions instead
- The count data are read-based or from a technology not matched by the fitted read-UMI and gamma priors; refit priors (nbinom.estimation, mixed.gamma.estimation) on matched pilot data first
- Cell annotations are continuous (e.g. pseudotime) and not yet discretized into cell types

## Inputs

- UMI count matrix (2D genes x cells), e.g. scPower's count.matrix.example ('complete', 'subsampled75', 'subsampled50', 'subsampled25' from GSE185714)
- Cell annotation data frame with a cell_type column, row order matching count matrix columns
- Fitted expression priors: read.umi.fit, gamma.linear.fit, and dispersion function (e.g. read.umi.fit.new, gamma.linear.fit.new, disp.fun.general.new)
- Experimental design parameters: nSamples (donors), cells per cell type, transcriptome.mapped.reads

## Outputs

- Predicted number of expressed genes per cell type (rounded sum of per-gene expression probabilities)
- Observed expressed gene counts from pseudobulk aggregation (calculate.gene.counts)
- Validation table (e.g. expression_probability_validation.tsv) with matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2
- Predicted-vs-observed plot against transcriptome.mapped.reads (ggplot2 with reshape2::melt)

## How to apply

Aggregate the 2D UMI count matrix into a 3D pseudobulk with scPower's create.pseudobulk (colName='cell_type') and count expressed genes with calculate.gene.counts using min.counts=3 and perc.indiv=0.5 to obtain observed expressed-gene numbers. Then run estimate.exp.prob.param with the planned nSamples (e.g. 14 donors), nCellsCt (num.cells/nDonors), the experiment-specific transcriptome.mapped.reads, and the fitted priors read.umi.fit.new, gamma.linear.fit.new (ct='New_ct') and disp.fun.general.new; sum the per-gene probabilities and round to get the predicted number of expressed genes. The rationale is that scPower models gene detection probability from a zero-inflated mixture of two left-censored gamma components fitted over mean expression values, so predictions reflect the joint effect of cell number and read depth. Compute r2 between predicted and observed counts across read-depth conditions (e.g. complete/subsampled75/50/25 of GSE185714 PBMC data) to quantify agreement.

## Related tools

- **scPower** (R package providing create.pseudobulk, calculate.gene.counts, estimate.exp.prob.param and the gamma-mixture prior fitting used to predict expressed gene counts) — https://github.com/heiniglab/scPower
- **R** (Runtime environment; scPower is an open source R package (installed via devtools::install_github)) — https://github.com/heiniglab/scPower
- **ggplot2** (Plotting predicted vs observed expressed gene counts against transcriptome.mapped.reads (with reshape2::melt))

## Examples

```
library(scPower); pb <- create.pseudobulk(count.matrix.example$complete, annot.df, colName='cell_type'); obs <- calculate.gene.counts(pb, min.counts=3, perc.indiv=0.5); est <- estimate.exp.prob.param(nSamples=14, nCellsCt=num.cells/14, transcriptome.mapped.reads=reads, read.umi.fit=read.umi.fit.new, gamma.linear.fit=gamma.linear.fit.new, ct='New_ct', disp.fun=disp.fun.general.new, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- r2 between predicted (estimated.genes) and observed (expressed.genes) counts across read-depth conditions exceeds ~0.9 on the training PBMC data
- Agreement holds at both detection cutoffs min.counts=3 with perc.indiv=0.5 evaluated at counts >10 and >0 thresholds
- Output table contains all required columns (matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2)
- Annotation data frame row order matches count matrix columns before pseudobulk aggregation
- Estimated gene numbers follow the vignette expectation that estimates 'match the observed numbers quite well' across subsampled read depths

## Limitations

- Data-driven priors only reliably assess power for sample sizes smaller or roughly equal to the pilot data set used for fitting
- Actual experiments may show higher doublet rates and fewer usable cells than assumed
- Toy example is limited to 14 donors and GSE185714-derived PBMC matrices; other platforms/tissues need refit priors

## Evidence

- [results] Subsampling one batch of the 10X PBMC training data at different read depths shows that the observed number of expressed genes per cell type is closely matched by the model expectation: [evidence span withheld — anchor status: not_found]
- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
- [methods] A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [methods] A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714): "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714)"
