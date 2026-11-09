---
name: read-depth-subsampling-comparison
description: Use when you have a UMI/read count matrix with donor and cell-type annotations
  and need to verify that scPower's predicted expressed-gene counts track observed
  counts across read-depth conditions before trusting power predictions.
when_to_use_negative:
- You need a forward-looking power/sample-size optimization for a new experiment rather
  than a model validation against observed data — use scPower's power-experiment functions
  instead.
- The input data lack donor/cell-type annotations or cannot be subsampled to multiple
  read depths, so no predicted-vs-observed comparison across conditions is possible.
- Your question concerns DEG/eQTL detection power or budget optimization, which requires
  effect-size priors and cost functions beyond expression-probability validation.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: scPower
  role: Provides create.pseudobulk, calculate.gene.counts, nbinom.estimation, mixed.gamma.estimation,
    estimate.exp.prob.param and toy GSE185714 example data for the subsampling comparison
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Implementation language for the validation workflow, including ggplot2/reshape2
    plotting and r2 computation
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:da92d53114cbd7ffd10d2e2afe39c1affe71a001dfbd3e543caf3bafee554787
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# read-depth-subsampling-comparison

## Summary

Validates the scPower expression probability model by subsampling a scRNA-seq count matrix at multiple read depths, then comparing predicted vs observed numbers of expressed genes per cell type (r2 agreement, e.g. >0.9 on PBMC toy data and ~0.99 for Drop-seq lung and Smart-seq2 pancreas).

## When to use

Use when you have a UMI/read count matrix with donor and cell-type annotations and need to verify that scPower's predicted expressed-gene counts track observed counts across read-depth conditions before trusting power predictions.

## When NOT to use

- You need a forward-looking power/sample-size optimization for a new experiment rather than a model validation against observed data — use scPower's power-experiment functions instead.
- The input data lack donor/cell-type annotations or cannot be subsampled to multiple read depths, so no predicted-vs-observed comparison across conditions is possible.
- Your question concerns DEG/eQTL detection power or budget optimization, which requires effect-size priors and cost functions beyond expression-probability validation.

## Inputs

- UMI/read count matrix (2D genes x cells) or scPower toy matrices count.matrix.example ('complete', 'subsampled75', 'subsampled50', 'subsampled25' from GSE185714)
- Annotation data frame with donor and cell_type columns, rows matching count matrix column order
- Fitted expression priors: read.umi.fit, gamma.linear.fit, dispersion function (e.g. disp.fun.general.new or disp.fun.param.smart)
- Per-matrix transcriptome.mapped.reads values and number of donors (e.g. nSamples=14)

## Outputs

- Table (e.g. expression_probability_validation.tsv / expression_model_validation_r2.csv) with columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes (observed), estimated.genes (predicted), r2
- ggplot2 plot of predicted vs observed expressed genes against transcriptome.mapped.reads (using reshape2::melt)

## How to apply

Load the count matrices (e.g. the scPower toy matrices 'complete', 'subsampled75', 'subsampled50', 'subsampled25' derived from GSE185714) with an annotation data frame whose row order matches the count matrix columns. Reformat each 2D matrix into a 3D pseudobulk with create.pseudobulk (colName='cell_type') and count expressed genes with calculate.gene.counts using min.counts=3 and perc.indiv=0.5 to get observed values. For each matrix, run estimate.exp.prob.param with the matrix-specific transcriptome.mapped.reads, the fitted read.umi.fit/gamma fits (e.g. read.umi.fit.new, gamma.linear.fit.new, disp.fun.general.new), nSamples donors, nCellsCt=num.cells/nDonors, and the same min.counts=3 / perc.indiv=0.5 thresholds, rounding summed probabilities for the predicted expressed-gene number. For non-10X technologies apply technology-specific adaptations: constant multiplet rate for Drop-seq (multipletRateGrowth='constant'), and for Smart-seq2 the gene-length normalized threshold (min.norm.count=3) with a read-depth-dependent dispersion fit (disp.fun.param). Compute r2 between predicted and observed counts across read depths (targets: PBMC >0.9, Drop-seq lung 0.995, Smart-seq2 pancreas 0.991, Supplementary Table S3) and write a table with matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2.

## Related tools

- **scPower** (Provides create.pseudobulk, calculate.gene.counts, nbinom.estimation, mixed.gamma.estimation, estimate.exp.prob.param and toy GSE185714 example data for the subsampling comparison) — https://github.com/heiniglab/scPower
- **R** (Implementation language for the validation workflow, including ggplot2/reshape2 plotting and r2 computation) — https://github.com/heiniglab/scPower

## Examples

```
pb <- scPower::create.pseudobulk(count.matrix.example$complete, annot.df, colName='cell_type'); obs <- scPower::calculate.gene.counts(pb, min.counts=3, perc.indiv=0.5)
```

## Evaluation signals

- r2 between predicted and observed expressed-gene counts exceeds 0.9 on the 10X PBMC toy subsampling series (counts>0 and counts>10 cutoffs).
- r2 reaches ~0.995 for Drop-seq lung data and ~0.991 for Smart-seq2 pancreas data, matching Supplementary Table S3.
- Output table contains exactly the columns matrix, num.cells, cells.indiv, transcriptome.mapped.reads, expressed.genes, estimated.genes, r2 with one row per read-depth condition.
- Annotation data frame row order matches count matrix column order (a required precondition before create.pseudobulk).
- Expressed-gene counting uses min.counts=3 and perc.indiv=0.5 consistently for both observed and predicted values.

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets.
- Actual experiments might result in higher doublet rates and lower number of usable cells than the default model assumptions.
- Non-UMI technologies like Smart-seq2 require technology-specific adaptations (gene-length normalized threshold min.norm.count, read-depth-dependent dispersion fit); the default UMI-based fits do not transfer directly.

## Evidence

- [methods] A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714): "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714)"
- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
- [readme] The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts.: "The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts."
