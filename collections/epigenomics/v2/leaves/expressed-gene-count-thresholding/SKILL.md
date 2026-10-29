---
name: expressed-gene-count-thresholding
description: Use when 2D single-cell UMI/read count matrices plus cell annotations
  (donor and cell type) must be converted to pseudobulk and the number of expressed
  genes per sample counted at specific per-cell thresholds, e.g. to validate expression
  probability models.
when_to_use_negative:
- Input is already pseudobulk or bulk expression data — create.pseudobulk aggregation
  per donor and cell type is unnecessary and inappropriate
- Goal is differential expression testing (edgeR/limma-voom pseudobulk DE) rather
  than quantifying how many genes are expressed above a threshold
- Count data are read-depth normalized transcripts rather than raw UMI counts, since
  thresholds like count > 0 / count > 10 are defined on per-pseudobulk UMI counts
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing create.pseudobulk (2D count matrix to 3D pseudobulk per
    donor/cell type) and calculate.gene.counts (extraction of expressed genes at a
    threshold)
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower functions are called for the validation
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:ef352497904e6b546338fc34e989d9dc7ed6e917326c4ba454175299c226a741
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - calculate.gene.counts
  - estimate.exp.prob.param
  - scPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# expressed-gene-count-thresholding

## Summary

Counts the number of expressed genes per pseudobulk sample (donor x cell type) from single-cell UMI count matrices, at user-defined expression thresholds (e.g. UMI count > 0 and count > 10), as implemented in scPower's calculate.gene.counts; these observed counts are used to benchmark predicted expression probabilities.

## When to use

Use when 2D single-cell UMI/read count matrices plus cell annotations (donor and cell type) must be converted to pseudobulk and the number of expressed genes per sample counted at specific per-cell thresholds, e.g. to validate expression probability models.

## When NOT to use

- Input is already pseudobulk or bulk expression data — create.pseudobulk aggregation per donor and cell type is unnecessary and inappropriate
- Goal is differential expression testing (edgeR/limma-voom pseudobulk DE) rather than quantifying how many genes are expressed above a threshold
- Count data are read-depth normalized transcripts rather than raw UMI counts, since thresholds like count > 0 / count > 10 are defined on per-pseudobulk UMI counts

## Inputs

- 2D UMI/read count matrices per batch (e.g. independent PBMC validation data, 25,000 reads per cell)
- Annotation data frame with donor and cell type per cell, row order matching count matrix columns
- Threshold value(s) in UMI counts per pseudobulk sample (e.g. 0 and 10)

## Outputs

- Observed number of expressed genes per batch and per threshold (count > 0 and count > 10)
- Per-batch table joined with predicted expressed gene counts, written to expressed_genes_validation.csv
- Predicted vs. observed expressed gene counts plot in the Fig. 2c layout

## How to apply

First reformat each batch's 2D count matrix into a 3D pseudobulk matrix per donor and cell type using scPower's create.pseudobulk, keeping annotation row ordering aligned with count matrix column ordering. Then extract expressed genes per pseudobulk sample with calculate.gene.counts at two thresholds: UMI count > 0 and count > 10 per pseudobulk sample, per batch (e.g. batches of 4, 4, and 8 individuals at 25,000 reads per cell). Join the observed expressed-gene counts per batch with model-predicted counts (from estimate.exp.prob.param summed over genes) and compute r2 between predicted and observed values. Correct application is judged by near-unity agreement, e.g. r2 > 0.9 across all batches as reported for the independent PBMC validation dataset.

## Related tools

- **scPower** (R package providing create.pseudobulk (2D count matrix to 3D pseudobulk per donor/cell type) and calculate.gene.counts (extraction of expressed genes at a threshold)) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower functions are called for the validation workflow)

## Examples

```
exp.genes <- calculate.gene.counts(pseudobulk.matrix, threshold = 10)
```

## Evaluation signals

- Predicted vs. observed expressed gene counts per batch yield r2 > 0.9 at both thresholds (count > 0 and count > 10)
- Annotation data frame row ordering matches count matrix column ordering before create.pseudobulk is called
- Two threshold levels (0 and 10 UMI counts per pseudobulk sample) are evaluated per batch, not just one
- Predicted counts are derived from estimate.exp.prob.param with readDepth=25000 and per-batch cells per individual, then summed over genes
- Per-batch table is written to expressed_genes_validation.csv and the plot matches the Fig. 2c layout

## Limitations

- Threshold choice matters: best-practice RNA-seq guidelines recommend cutoffs removing 19-33% of lowly expressed genes (1-10 reads per million / 1-5 UMI counts), so the count > 0 vs. count > 10 comparison brackets a practical range
- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets, so extrapolating observed counts far beyond the estimation data is unreliable

## Evidence

- [methods] Creation of pseudobulk matrix from count matrix: "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] Extraction of expressed genes at thresholds: "the expressed genes are extracted using the function "calculate.gene.counts""
- [results] Validation agreement criterion from the source task card: [evidence span withheld — anchor status: not_found]
- [readme] README points to the vignette reproducing the paper's plots: [evidence span withheld — anchor status: external_doc]
