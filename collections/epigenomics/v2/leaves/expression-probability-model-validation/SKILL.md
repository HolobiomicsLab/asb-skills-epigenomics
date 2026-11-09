---
name: expression-probability-model-validation
description: Use when a planned or pilot scRNA-seq design needs its predicted expressed-gene
  numbers checked against real data, e.g. subsampled UMI count matrices with per-cell-type
  annotations available.
when_to_use_negative:
- Input is a 3d pseudobulk matrix already, or expression priors must be fitted from
  scratch (use nbinom.estimation/mixed.gamma.estimation workflow instead)
- The goal is DEG/eQTL power or budget optimization rather than validating expression
  probability predictions against observed data
- Data are read-count based or from a non-droplet technology for which the GSE185714-derived
  UMI priors do not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPower
  role: R package providing create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param
    used to compute observed and predicted expressed-gene numbers
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which the validation script runs, with ggplot2 and reshape2 for
    plotting
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:fe1d2028586f78b9c2fa4f8e35498748c2f07cf3878687bcaaf1f7e831190580
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# expression-probability-model-validation

## Summary

Validates scPower's expression probability model by comparing predicted numbers of expressed genes per cell type against observed counts from subsampled 10X PBMC UMI count matrices, quantifying agreement with r2 across read-depth conditions.

## When to use

Use when a planned or pilot scRNA-seq design needs its predicted expressed-gene numbers checked against real data, e.g. subsampled UMI count matrices with per-cell-type annotations available.

## When NOT to use

- Input is a 3d pseudobulk matrix already, or expression priors must be fitted from scratch (use nbinom.estimation/mixed.gamma.estimation workflow instead)
- The goal is DEG/eQTL power or budget optimization rather than validating expression probability predictions against observed data
- Data are read-count based or from a non-droplet technology for which the GSE185714-derived UMI priors do not apply

## Inputs

- UMI count matrices (count.matrix.example: 'complete', 'subsampled75', 'subsampled50', 'subsampled25' derived from GSE185714)
- Cell annotation data frame (annot.df) with row order matching count matrix columns, including a 'cell_type' column
- Fitted scPower priors: read.umi.fit.new, gamma.linear.fit.new (ct='New_ct'), disp.fun.general.new
- Per-matrix parameters: number of cells, cells per individual, transcriptome.mapped.reads, nSamples=14 donors

## Outputs

- expression_probability_validation.tsv with columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes (observed), estimated.genes (predicted), r2
- ggplot2 plot of predicted vs observed expressed-gene counts against transcriptome.mapped.reads (using reshape2::melt)
- r2 statistic quantifying model agreement across read-depth conditions

## How to apply

Load toy UMI count matrices (count.matrix.example: 'complete', 'subsampled75', 'subsampled50', 'subsampled25' from GSE185714) plus an annotation data frame whose rows match count-matrix columns. For each matrix, aggregate to a 3d pseudobulk with create.pseudobulk (colName='cell_type') and count expressed genes via calculate.gene.counts with min.counts=3 and perc.indiv=0.5 to get observed values. Then run estimate.exp.prob.param with nSamples=14 donors, nCellsCt=num.cells/nDonors, matrix-specific transcriptome.mapped.reads, the fitted read.umi.fit.new and gamma.linear.fit.new (ct='New_ct') and disp.fun.general.new, same min.counts/perc.indiv, rounding summed probabilities to predicted expressed-gene numbers. Compute r2 between predicted and observed counts across the read-depth conditions; the model is accepted when estimated numbers match the observed numbers 'quite well' (r2 > 0.9) at both count>0 and count>10 thresholds.

## Related tools

- **scPower** (R package providing create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param used to compute observed and predicted expressed-gene numbers) — https://github.com/heiniglab/scPower
- **R** (Language in which the validation script runs, with ggplot2 and reshape2 for plotting) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); pseudobulk <- create.pseudobulk(count.matrix, annot.df, colName='cell_type'); obs <- calculate.gene.counts(pseudobulk, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- r2 between predicted and observed expressed-gene counts exceeds 0.9 across the read-depth conditions
- Predicted expressed-gene numbers closely match observed values at both count>10 and count>0 cutoffs
- Output TSV contains exactly the columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2
- Observed counts come from pseudobulk aggregation with min.counts=3 and perc.indiv=0.5, matching the parameters passed to estimate.exp.prob.param
- annot.df row order matches count matrix column order (mismatch would invalidate observed counts)

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Validation relies on the toy GSE185714-derived subsampled matrices and fitted priors (read.umi.fit.new, gamma.linear.fit.new, disp.fun.general.new); new tissues/technologies require refitting priors
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the model assumes

## Evidence

- [results] Subsampling one batch of the 10X PBMC training data: [evidence span withheld — anchor status: not_found]
- [methods] pseudobulk creation: "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] gene counting function: "the expressed genes are extracted using the function "calculate.gene.counts""
- [methods] toy dataset provenance: "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714)"
- [readme] vignette availability: "A short tutorial is given in the vignette [introduction-scPower](vignettes/introduction-scPower.pdf)."
