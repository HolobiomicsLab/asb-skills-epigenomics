---
name: expression-probability-prediction
description: Use when designing or validating single cell RNA-seq experiments and
  you need per-gene detection probabilities at a given read depth, cells per individual,
  and sample size, or when checking that model-predicted expressed gene counts match
  observed counts in a held-out dataset.
when_to_use_negative:
- The goal is differential expression or eQTL power itself rather than the per-gene
  expression detection probability component — use the combined scPower power estimation
  that multiplies expression probability with eQTL power instead
- The pilot data used to fit priors comes from a different tissue or platform technology
  than the target counts (priors are UMI/read-count and tissue specific)
- Continuous cell annotations (e.g. pseudo time) are present and cannot be discretized
  into donor/cell type categories required by create.pseudobulk
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPower
  role: R package providing create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param
    to build pseudobulk matrices, count observed expressed genes, and predict per-gene
    expression probabilities from fitted priors
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language in which scPower is implemented and in which the validation workflow
    (join predicted/observed counts, compute r2, export CSV, plot Fig. 2c layout)
    is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:b726a5e8027ff1550997dffe5edae6c4652a38398e398cfc045bfc245e48b994
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - estimate.exp.prob.param
  - scPower
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# expression-probability-prediction

## Summary

Predict per-gene cell-type-specific expression probabilities for a scRNA-seq design using scPower's parametric model (gamma mixed expression priors, read-UMI and mean-dispersion fits), and validate predictions by comparing summed predicted versus observed expressed-gene counts (r2 > 0.9) against independent pseudobulk data.

## When to use

Use when designing or validating single cell RNA-seq experiments and you need per-gene detection probabilities at a given read depth, cells per individual, and sample size, or when checking that model-predicted expressed gene counts match observed counts in a held-out dataset.

## When NOT to use

- The goal is differential expression or eQTL power itself rather than the per-gene expression detection probability component — use the combined scPower power estimation that multiplies expression probability with eQTL power instead
- The pilot data used to fit priors comes from a different tissue or platform technology than the target counts (priors are UMI/read-count and tissue specific)
- Continuous cell annotations (e.g. pseudo time) are present and cannot be discretized into donor/cell type categories required by create.pseudobulk

## Inputs

- UMI count matrix (2D genes x cells) per batch, e.g. independent PBMC validation batches with 4, 4, and 8 individuals at 25,000 reads per cell
- Annotation data frame mapping each cell to donor and cell type, with row ordering matching count matrix column ordering
- Expression priors estimated on a pilot dataset: gamma mixed distribution fits over mean values, read-UMI fit, and per-gene negative binomial mean-dispersion fits (nbinom.estimation)

## Outputs

- Per-gene expression probability table per batch/design at the specified readDepth and cells per individual
- Predicted vs observed expressed gene counts per batch at count>0 and count>10 thresholds, exported as expressed_genes_validation.csv
- r2 values between predicted and observed expressed gene counts per batch and threshold (expected > 0.9)
- Predicted-vs-observed scatter plot per batch in the Fig. 2c layout

## How to apply

Load the UMI count matrix plus the annotation data frame (donor and cell type per cell) for each batch in R with the scPower package, and reformat the 2D count matrix into a 3D pseudobulk matrix per donor and cell type with create.pseudobulk, keeping annotation row ordering aligned with count matrix column ordering. Count observed expressed genes per batch with calculate.gene.counts at two thresholds (count>0 and count>10 per pseudobulk sample). Predict per-gene expression probabilities with estimate.exp.prob.param at the target read depth (e.g. 25,000 reads per cell) and per-batch cells-per-individual, supplying the gamma mixed fits, the read-UMI fit, and the mean-dispersion (negative binomial) fits learned from a pilot dataset that excludes the validation data; sum probabilities to get predicted expressed gene counts. Join predicted and observed counts per batch per threshold and compute r2; on the independent PBMC validation set (batches A/B with 4 individuals, batch C with 8 individuals at 25,000 reads/cell) all r2 exceeded 0.9, and results are written to expressed_genes_validation.csv and plotted predicted-vs-observed per batch (Fig. 2c layout). The rationale is that overall detection power is the product of the expression probability of a gene and its eQTL/DE power, so accurate expression probability estimation underpins the whole power model.

## Related tools

- **scPower** (R package providing create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param to build pseudobulk matrices, count observed expressed genes, and predict per-gene expression probabilities from fitted priors) — https://github.com/heiniglab/scPower
- **R** (Language in which scPower is implemented and in which the validation workflow (join predicted/observed counts, compute r2, export CSV, plot Fig. 2c layout) is executed)

## Examples

```
prob <- estimate.exp.prob.param(gamma.mixed.fits=gamma.fits, read.umi.fit=read.umi.fit, gamma.lin.UD=FALSE, readDepth=25000, cells.individual=cellsPerIndividual, ct="PBMC", dispersion.specified=TRUE, return.parameters=FALSE); n.expressed.pred <- colSums(prob$prob.exp)
```

## Evaluation signals

- r2 between predicted and observed expressed gene counts exceeds 0.9 for every batch at both thresholds (count>0 and count>10)
- Predicted expressed gene counts closely match observed numbers of expressed genes on the independent PBMC validation data at 25,000 reads per cell
- Annotation data frame row ordering is verified to align with count matrix column ordering before create.pseudobulk
- Output file expressed_genes_validation.csv contains one row per batch per threshold with paired predicted and observed counts
- Predicted probabilities are summed per batch and exceed (are consistent with) the observed counts obtained from calculate.gene.counts

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the sample size of the pilot data sets
- Expression probability prediction depends on priors (gamma mixed fits, read-UMI fit, mean-dispersion fit) learned without the validation data; mismatched priors degrade r2
- Validation in the paper used read-count-based PBMC batches at a fixed 25,000 reads per cell with 4/4/8 individuals; other depths, technologies, or sample sizes require re-estimating parameters

## Evidence

- [methods] pseudobulk creation: "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] expressed gene counting: "the expressed genes are extracted using the function "calculate.gene.counts""
- [methods] power decomposition: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
- [methods] negative binomial fits: "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
- [readme] package availability: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
