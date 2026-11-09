---
name: r-squared-goodness-of-fit-assessment
description: Use when a parametric model (e.g. scPower expression probability model)
  yields numeric predictions that must be validated against observed values across
  controlled conditions such as sequencing-depth subsampling of count matrices.
when_to_use_negative:
- When the goal is experimental design optimization (budget-constrained parameter
  search with scPower) rather than validating model predictions against observed data
- When predicted and observed values are not paired across comparable conditions (e.g.
  a single sample with no read-depth series), making r2 over conditions meaningless
- When input data are not scRNA-seq count matrices with matching cell-type annotations
  (e.g. feature tables or bulk RNA-seq without a pseudobulk aggregation step)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scPower
  role: R package supplying create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param
    to generate observed and predicted expressed gene counts for the r2 comparison
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which the scPower model, fitting, r2 computation, and ggplot2
    validation plot are executed
- name: reshape2
  role: Used with melt to reshape predicted/observed values for the ggplot2 validation
    plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:69de8cea5dd12a57a9b168db536867b91216ed2f5fe188bae23a0eee9eb833d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - reshape2
  - scPower
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-squared-goodness-of-fit-assessment

## Summary

Quantifies agreement between model-predicted and observed quantities by computing the r-squared across paired conditions — here, scPower-estimated expressed gene numbers versus observed expressed gene counts from subsampled 10X PBMC UMI count matrices at different read depths.

## When to use

Use when a parametric model (e.g. scPower expression probability model) yields numeric predictions that must be validated against observed values across controlled conditions such as sequencing-depth subsampling of count matrices.

## When NOT to use

- When the goal is experimental design optimization (budget-constrained parameter search with scPower) rather than validating model predictions against observed data
- When predicted and observed values are not paired across comparable conditions (e.g. a single sample with no read-depth series), making r2 over conditions meaningless
- When input data are not scRNA-seq count matrices with matching cell-type annotations (e.g. feature tables or bulk RNA-seq without a pseudobulk aggregation step)

## Inputs

- 2d UMI count matrices (count.matrix.example: 'complete', 'subsampled75', 'subsampled50', 'subsampled25' from GSE185714)
- Cell annotation data frame (annot.df) with row order matching count matrix columns
- Fitted scPower priors: read.umi.fit.new, gamma.linear.fit.new, disp.fun.general.new
- Per-matrix design parameters: num.cells, cells.indiv, transcriptome.mapped.reads

## Outputs

- expression_probability_validation.tsv with columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes (observed), estimated.genes (predicted), r2
- Predicted-vs-observed plot against transcriptome.mapped.reads (ggplot2/reshape2::melt)
- r2 value quantifying model-observation agreement across read-depth conditions

## How to apply

Generate paired predicted and observed values across conditions: for each UMI count matrix (e.g. 'complete', 'subsampled75', 'subsampled50', 'subsampled25' from GSE185714), aggregate counts into a 3d pseudobulk with scPower's create.pseudobulk (colName='cell_type') and count expressed genes via calculate.gene.counts with min.counts=3 and perc.indiv=0.5 to get observed values. Obtain predictions by running estimate.exp.prob.param (nSamples=14 donors, nCellsCt=num.cells/nDonors, matrix-specific transcriptome.mapped.reads, fitted read.umi.fit.new, gamma.linear.fit.new with ct='New_ct', disp.fun.general.new) and rounding the summed probabilities. Compute r2 between predicted and observed counts both across all read-depth conditions and per condition, write a TSV with columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2 (expression_probability_validation.tsv), and plot predicted vs observed against transcriptome.mapped.reads using ggplot2 with reshape2::melt.

## Related tools

- **scPower** (R package supplying create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param to generate observed and predicted expressed gene counts for the r2 comparison) — https://github.com/heiniglab/scPower
- **R** (Language in which the scPower model, fitting, r2 computation, and ggplot2 validation plot are executed)
- **reshape2** (Used with melt to reshape predicted/observed values for the ggplot2 validation plot)

## Examples

```
pb.3d <- create.pseudobulk(count.matrix.example$complete, annot.df, colName="cell_type"); obs <- calculate.gene.counts(pb.3d, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- r2 between predicted (estimated) and observed expressed gene counts exceeds 0.9 on the training PBMC data
- Agreement holds for both expression thresholds, counts >10 and counts >0
- expression_probability_validation.tsv contains all required columns (matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2) with one row per subsampled matrix
- annot.df row order was verified to match count matrix columns before pseudobulk aggregation
- Estimated gene numbers were obtained by rounding summed expression probabilities from estimate.exp.prob.param with min.counts=3 and perc.indiv=0.5

## Limitations

- Data driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Validation is demonstrated on a small toy data set subsampled from a single batch (GSE185714, 14 donors); actual experiments might result in higher doublet rates and lower numbers of usable cells than assumed
- scPower was developed for pseudobulk-based interindividual DEG/eQTL power analysis; r2 agreement on expressed gene counts does not guarantee power calibration in other designs

## Evidence

- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
- [results] Subsampling one batch of the 10X PBMC training data at different read depths shows that the observed number of expressed genes per cell type is closely matched by the model expectation: [evidence span withheld — anchor status: not_found]
- [other] the estimates match the observed numbers quite well: [evidence span withheld — anchor status: not_found]
- [readme] A short tutorial is given in the vignette [introduction-scPower](vignettes/introduction-scPower.pdf).: "A short tutorial is given in the vignette [introduction-scPower](vignettes/introduction-scPower.pdf)."
