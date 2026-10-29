---
name: pseudobulk-count-matrix-construction
description: Use when a per-cell count matrix (UMI or read counts) plus donor/sample
  and cell-type annotations must be aggregated into pseudobulk form to fit expression
  priors or run pseudobulk-based DEG/eQTL power analysis with scPower.
when_to_use_negative:
- Input is already a pseudobulk/aggregated gene-by-sample expression matrix rather
  than a per-cell count matrix
- The analysis goal is single-cell-level modeling or simulation with tools like muscat
  or powsimR rather than the pseudobulk approach
- No donor/sample or cell-type annotation is available, so the 2D-to-3D reformatting
  cannot be performed reliably
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation,
    meanUMI.calculation, and convert.gamma.parameters for pseudobulk construction
    and expression prior fitting
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Implementation environment; scPower is distributed as an open source R package
    installable via devtools
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:15d4bbac93846b30b61b864a7479d103f3cf115e434bf92ff87ac5e7180221c0
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pseudobulk-count-matrix-construction

## Summary

Reformat a 2D single-cell UMI/read count matrix into a 3D pseudobulk matrix (genes × samples × cell types) using scPower's create.pseudobulk, the entry step for scPower's expression probability model, which then estimates per-gene negative binomial mean/dispersion and gamma-mixed expression priors from the aggregated counts.

## When to use

Use when a per-cell count matrix (UMI or read counts) plus donor/sample and cell-type annotations must be aggregated into pseudobulk form to fit expression priors or run pseudobulk-based DEG/eQTL power analysis with scPower.

## When NOT to use

- Input is already a pseudobulk/aggregated gene-by-sample expression matrix rather than a per-cell count matrix
- The analysis goal is single-cell-level modeling or simulation with tools like muscat or powsimR rather than the pseudobulk approach
- No donor/sample or cell-type annotation is available, so the 2D-to-3D reformatting cannot be performed reliably

## Inputs

- 2D single-cell count matrix (UMI counts for droplet-based data, read counts for Smart-seq2 data)
- Annotation data frame with donor/sample and cell-type labels aligned to count matrix columns
- Read-depth subsampled count matrices (for technology generalization experiments)

## Outputs

- 3D pseudobulk count matrix (genes × samples × cell types)
- Per-gene negative binomial mean and dispersion estimates (nbinom.estimation)
- Gamma mixed distribution fits over mean values (mixed.gamma.estimation, num.genes.kept=21000)

## How to apply

Load the atlas count matrices together with the annotation data frames, ensuring annotation rows match count matrix column ordering, then call scPower's create.pseudobulk to reformat the 2D count matrix into a 3D pseudobulk matrix. From the pseudobulk matrix, estimate per-gene negative binomial mean and dispersion with nbinom.estimation (DESeq-based normalization) for each read-depth subsample. Fit a gamma mixed distribution (a zero component plus two left-censored gamma components) over the mean value vectors with mixed.gamma.estimation (num.genes.kept=21000, censoredPoint=1/num_cells_per_celltype). For droplet-based (UMI) data, additionally compute mean UMI per cell with meanUMI.calculation and convert gamma rate-shape parameters to mean-sd parameterization via convert.gamma.parameters; the resulting priors support technology-specific adaptations (constant doublet rate for Drop-seq, gene-length-normalized thresholds for Smart-seq2).

## Related tools

- **scPower** (R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation, meanUMI.calculation, and convert.gamma.parameters for pseudobulk construction and expression prior fitting) — https://github.com/heiniglab/scPower
- **R** (Implementation environment; scPower is distributed as an open source R package installable via devtools) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); pseudobulk <- create.pseudobulk(count.matrix, annotation, sample.label="sample", celltype.label="cell_type")
```

## Evaluation signals

- The resulting object is a 3D pseudobulk matrix whose sample/cell-type dimension ordering exactly matches the annotation data frame rows
- nbinom.estimation returns finite mean and dispersion parameters for each gene without errors from DESeq-based normalization
- mixed.gamma.estimation succeeds with num.genes.kept=21000 and censoredPoint=1/num_cells_per_celltype, and fits can be inspected with visualize.gamma.fits
- Downstream, predicted vs observed expressed gene counts (via calculate.gene.counts and estimate.exp.prob.param with min.counts=3, perc.indiv=0.5) reach r2 ≈ 0.995 for Drop-seq lung and r2 ≈ 0.991 for Smart-seq2 pancreas (Supplementary Table S3)

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Annotation rows must match count matrix column ordering; misalignment silently corrupts the pseudobulk aggregation
- Actual experiments might result in higher doublet rates and lower number of usable cells than the default assumption

## Evidence

- [methods] Creation of pseudobulk matrix from count matrix: "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] Negative binomial parameter estimation per gene: "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
- [methods] Gamma mixed distribution fit: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [readme] Tool evaluation across technologies: "The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts."
