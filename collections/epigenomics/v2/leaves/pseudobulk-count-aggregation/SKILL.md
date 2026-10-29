---
name: pseudobulk-count-aggregation
description: Use when a per-cell UMI/read count matrix plus a cell annotation data
  frame (donor and cell type per cell) must be converted to donor-by-cell-type pseudobulk
  matrices to count expressed genes (e.g. count>0 and count>10) for power modelling
  or model validation.
when_to_use_negative:
- The goal is fitting expression priors from raw data (nbinom.estimation / mixed.gamma.estimation),
  not aggregating existing counts into pseudobulk
- Analysis is single-cell-level differential expression or simulation-based power
  analysis (e.g. muscat, powsimR) rather than pseudobulk-based scPower power estimation
- The input is already a pseudobulk (3D) matrix or a sample-level count table — re-aggregation
  would double-sum counts
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: Provides create.pseudobulk (2D count matrix to 3D pseudobulk matrix) and calculate.gene.counts
    (extraction of expressed genes per pseudobulk sample)
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Language environment in which the scPower functions are called to load count
    matrices, aggregate, and compute observed expressed gene counts
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:d8c453aeb2b71102cb286c872adbddfc06a5bc5f18dc76b07090b6c758c61e82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - create.pseudobulk
  - estimate.exp.prob.param
  - scPower
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pseudobulk-count-aggregation

## Summary

Aggregates a 2D single-cell UMI count matrix into a 3D pseudobulk matrix (gene x donor x cell type) with scPower's create.pseudobulk, then extracts expressed-gene counts per pseudobulk sample with calculate.gene.counts at defined count thresholds. This is the scPower-style prerequisite step for expression probability modelling and validation of predicted vs observed expressed gene numbers.

## When to use

Use when a per-cell UMI/read count matrix plus a cell annotation data frame (donor and cell type per cell) must be converted to donor-by-cell-type pseudobulk matrices to count expressed genes (e.g. count>0 and count>10) for power modelling or model validation.

## When NOT to use

- The goal is fitting expression priors from raw data (nbinom.estimation / mixed.gamma.estimation), not aggregating existing counts into pseudobulk
- Analysis is single-cell-level differential expression or simulation-based power analysis (e.g. muscat, powsimR) rather than pseudobulk-based scPower power estimation
- The input is already a pseudobulk (3D) matrix or a sample-level count table — re-aggregation would double-sum counts

## Inputs

- 2D UMI/read count matrix per batch (genes x cells)
- Cell annotation data frame with donor (individual) and cell type per cell
- Batch metadata: number of individuals per batch and sequencing depth (e.g. 25,000 reads per cell)

## Outputs

- 3D pseudobulk matrix (gene x donor x cell type) per batch
- Observed number of expressed genes per batch at the two thresholds (count>0 and count>10)
- expressed_genes_validation.csv with per-batch predicted vs observed expressed gene counts (Fig. 2c layout plot)

## How to apply

Load each batch's 2D UMI count matrix and its annotation data frame in R with the scPower package, keeping the annotation row ordering exactly aligned with the count matrix column ordering. Call scPower's create.pseudobulk to reformat the 2D matrix into a 3D gene x donor x cell type pseudobulk matrix per batch (e.g. batches of 4, 4, and 8 individuals at 25,000 reads per cell). Then run calculate.gene.counts on the pseudobulk object to extract the number of expressed genes per batch at both thresholds (count>0 and count>10 per pseudobulk sample). These observed counts are what scPower compares against per-gene expression probabilities summed from estimate.exp.prob.param, and validation succeeds when predicted vs observed r2 > 0.9 per batch.

## Related tools

- **scPower** (Provides create.pseudobulk (2D count matrix to 3D pseudobulk matrix) and calculate.gene.counts (extraction of expressed genes per pseudobulk sample)) — https://github.com/heiniglab/scPower
- **R** (Language environment in which the scPower functions are called to load count matrices, aggregate, and compute observed expressed gene counts) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); pb <- create.pseudobulk(count.matrix = counts, annotation = anno, name.column = c("donor", "celltype")); gene.counts <- calculate.gene.counts(pb, expr.cutoff = 10)
```

## Evaluation signals

- Annotation row ordering is aligned with count matrix column ordering before create.pseudobulk is called (no donor/cell type misassignment)
- Pseudobulk matrix has 3 dimensions (gene x donor x cell type) with one entry per donor per cell type per batch
- Observed expressed gene counts are reported at both thresholds count>0 and count>10 per pseudobulk sample
- Predicted vs observed expressed gene counts per batch yield r2 > 0.9 (e.g. batches A/B with 4 individuals, batch C with 8 individuals at 25,000 reads per cell)
- Per-batch table written to expressed_genes_validation.csv and plottable in the Fig. 2c predicted-vs-observed layout

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets, so validation/design beyond pilot scale is extrapolation
- Continuous cell annotations such as pseudo time would need to be discretized before the power analysis, so pseudobulk aggregation assumes categorical donor/cell type labels
- The model was evaluated on UMI counts and read counts from specific tissues and technologies; mismatched platforms may not generalize

## Evidence

- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
- [readme] scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq.: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
- [intro] the pseudobulk approach in combination with classical differential gene expression methods such as edgeR and limma-voom outperforms single cell specific methods: "the “pseudobulk” approach in combination with classical differential gene expression methods such as edgeR44 and limma-voom45 outperforms single cell specific methods"
