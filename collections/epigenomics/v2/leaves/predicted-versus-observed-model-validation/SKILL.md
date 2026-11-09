---
name: predicted-versus-observed-model-validation
description: Use when a fitted statistical model (e.g., expression probability priors
  from a pilot scRNA-seq study) must be shown to generalize to an independent dataset
  not used for parameter estimation, by regressing predicted against observed counts.
when_to_use_negative:
- The validation dataset was part of the data used to estimate the expression priors
  — comparing predictions on training data inflates r2 and does not test generalization
- The goal is power/design optimization rather than model validation — use scPower's
  experimental design functions instead
- Input is raw single-cell data without cell type and donor annotations, so pseudobulk
  aggregation per donor and cell type is not possible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package supplying create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param
    to build pseudobulk matrices, count observed expressed genes, and predict per-gene
    expression probabilities on the validation batches
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Environment in which the scPower validation workflow (pseudobulk creation,
    gene counting, probability prediction, r2 computation, and CSV/plot export) is
    executed
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:2ea991896e5c86d6ab527dcda65e9e0ac119565e2e9db8b7c41e1c27af286d7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - estimate.exp.prob.param
  - scPower
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# predicted-versus-observed-model-validation

## Summary

Validate a parametric expression probability model by comparing per-batch predicted expressed-gene counts against observed counts on an independent dataset, confirming generalization via r2 > 0.9. This is the scPower-style held-out validation of model priors learned without the validation data.

## When to use

Use when a fitted statistical model (e.g., expression probability priors from a pilot scRNA-seq study) must be shown to generalize to an independent dataset not used for parameter estimation, by regressing predicted against observed counts.

## When NOT to use

- The validation dataset was part of the data used to estimate the expression priors — comparing predictions on training data inflates r2 and does not test generalization
- The goal is power/design optimization rather than model validation — use scPower's experimental design functions instead
- Input is raw single-cell data without cell type and donor annotations, so pseudobulk aggregation per donor and cell type is not possible

## Inputs

- Independent validation PBMC UMI count matrices for three batches (batches A/B with 4 individuals, batch C with 8 individuals at 25,000 reads per cell)
- Annotation data frame with donor and cell type per cell, row-ordered to match count matrix columns
- Model parameters (gamma mixed fits, read-UMI fit, mean-dispersion fit) estimated without the validation data

## Outputs

- expressed_genes_validation.csv with per-batch predicted vs observed expressed gene counts at count>0 and count>10 thresholds
- Predicted vs observed scatter plot per batch in the Fig. 2c layout
- Per-batch r2 values (all expected to exceed 0.9)

## How to apply

Reformat each validation batch's 2D UMI count matrix into a 3D pseudobulk matrix per donor and cell type using scPower's create.pseudobulk, keeping annotation row ordering aligned with count matrix column ordering. Count observed expressed genes per batch with calculate.gene.counts at two thresholds (count>0 and count>10 per pseudobulk sample). Predict per-gene expression probabilities per batch with estimate.exp.prob.param (readDepth=25000, per-batch cells per individual, gamma mixed fits, read-UMI fit and mean-dispersion fit learned without the validation data) and sum probabilities to obtain predicted expressed gene counts. Join predicted and observed counts per batch for both thresholds and compute r2 between them; success criterion is r2 > 0.9 across all batches. Write the per-batch table to expressed_genes_validation.csv and plot predicted vs observed per batch in the Fig. 2c layout.

## Related tools

- **scPower** (R package supplying create.pseudobulk, calculate.gene.counts, and estimate.exp.prob.param to build pseudobulk matrices, count observed expressed genes, and predict per-gene expression probabilities on the validation batches) — https://github.com/heiniglab/scPower
- **R** (Environment in which the scPower validation workflow (pseudobulk creation, gene counting, probability prediction, r2 computation, and CSV/plot export) is executed) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); pbulk <- create.pseudobulk(count.matrix, annotation, sample.label="donor", cell.type.label="celltype"); obs <- calculate.gene.counts(pbulk, expressed.threshold=0)
```

## Evaluation signals

- All three validation batches (A, B, C) yield r2 > 0.9 between predicted and observed expressed gene counts
- Observed numbers of expressed genes at both thresholds (count>0 and count>10 per pseudobulk sample) are closely matched by model expectations
- Annotation data frame row ordering matches count matrix column ordering before create.pseudobulk aggregation
- expressed_genes_validation.csv contains joined predicted and observed counts for both thresholds for each of the three batches
- Predicted probabilities are summed per gene across the batch to produce a single predicted expressed-gene count per batch

## Limitations

- Data driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets
- Validation is specific to the read depth used in the prior estimation (here 25,000 reads per cell); extrapolation to very different depths is not tested by this workflow
- Larger sample sizes identify new significant DEGs with lower effect sizes not captured in pilot-study-based computation

## Evidence

- [results] On the independent PBMC validation data set (25,000 reads per cell; batches A/B with 4 individuals, batch C with 8 individuals), the observed numbers of expressed genes are closely matched by the model's expectations, with all r2 > 0.9.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step on pseudobulk creation: "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] Workflow step on observed gene counts: "the expressed genes are extracted using the function "calculate.gene.counts""
- [readme] README on package installation and vignettes for reproducing paper plots: [evidence span withheld — anchor status: external_doc]
- [intro] Article claim about model purpose framing the validation target: "including a quantification of the probability to detect cell type specific gene expression in scRNA-seq data"
