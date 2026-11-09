---
name: pseudobulk-differential-expression-testing-edger
description: 'Use when testing differential expression between individuals/groups
  from single cell RNA-seq UMI count data: the pseudobulk aggregation + edgeR route
  is the recommended analysis whose power scPower estimates.'
when_to_use_negative:
- When the goal is single-cell-level (within-individual, per-cell) differential testing
  rather than interindividual sample-level comparisons
- When counts are already aggregated to bulk per-sample profiles — 'create.pseudobulk'
  is unnecessary and cell type information would be lost
- When the analysis requires eQTL-level power design questions that need scPower's
  analytic model rather than a single DE run
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: Provides 'create.pseudobulk' to reformat the 2D count matrix into a 3D pseudobulk
    matrix; scPower's power model is designed around this exact analysis
  repo: https://github.com/heiniglab/scPower
- name: edgeR
  role: Performs the differential expression testing on the pseudobulk profiles (edgeR-LRT)
    and downstream evaluation, e.g. in the powsimR/muscat comparison simulations
- name: R
  role: Implementation environment for scPower (R package) and the pseudobulk DE workflow
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:a8ad8f7433c56836985ee806782c8dafe643ddf85d0f917670719c6bcd65ba37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - edgeR
  - muscat
  - powsimR
  - scPower
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Pseudobulk differential expression testing with edgeR

## Summary

Aggregates single cell UMI count matrices into per-sample pseudobulk profiles and applies classical bulk RNA-seq differential expression testing (edgeR-LRT) to interindividual cell type comparisons, the strategy the scPower power model is built around and which outperforms single-cell-specific methods.

## When to use

Use when testing differential expression between individuals/groups from single cell RNA-seq UMI count data: the pseudobulk aggregation + edgeR route is the recommended analysis whose power scPower estimates.

## When NOT to use

- When the goal is single-cell-level (within-individual, per-cell) differential testing rather than interindividual sample-level comparisons
- When counts are already aggregated to bulk per-sample profiles — 'create.pseudobulk' is unnecessary and cell type information would be lost
- When the analysis requires eQTL-level power design questions that need scPower's analytic model rather than a single DE run

## Inputs

- 2D single cell UMI count matrix (gene x cell)
- Per-cell sample/individual and cell type annotations
- Expressed-gene filter cutoffs (e.g. 1-5 UMI counts / 1-10 RPM)

## Outputs

- 3D pseudobulk count matrix (gene x sample x cell type)
- Per-gene differential expression test statistics and p-values from edgeR-LRT
- Bonferroni-corrected significant DEG calls at the 0.05 threshold

## How to apply

First reformat the 2D gene-by-cell UMI count matrix into a 3D pseudobulk matrix using scPower's 'create.pseudobulk' function, summing counts per gene, per cell type, and per sample. Analyze the pseudobulk profiles with edgeR, using the LRT test (edgeR-LRT) for differential expression between the conditions of interest. Apply a 0.05 significance threshold with Bonferroni correction based on the estimated number of expressed genes; the article notes best-practice cutoffs remove 19-33% of lowly expressed genes (roughly 1-10 reads per million / 1-5 UMI counts) before expression analysis. The rationale is that pseudobulk profiles restore the sample-level replication that classical bulk methods such as edgeR and limma-voom require, so these methods outperform single-cell-specific models and mixed models on interindividual comparisons. This same analysis pipeline (edgeR-LRT testing plus edgeR downstream evaluation) is what scPower's analytic power estimates of DE power and overall detection power are calibrated against, as validated by powsimR and muscat simulations.

## Related tools

- **scPower** (Provides 'create.pseudobulk' to reformat the 2D count matrix into a 3D pseudobulk matrix; scPower's power model is designed around this exact analysis) — https://github.com/heiniglab/scPower
- **edgeR** (Performs the differential expression testing on the pseudobulk profiles (edgeR-LRT) and downstream evaluation, e.g. in the powsimR/muscat comparison simulations)
- **R** (Implementation environment for scPower (R package) and the pseudobulk DE workflow) — https://github.com/heiniglab/scPower

## Examples

```
Rscript -e "library(scPower); pb <- create.pseudobulk(umi.counts, samples, celltypes)"  # then test pb per cell type with edgeR-LRT
```

## Evaluation signals

- The pseudobulk matrix must be 3D (gene x sample x cell type) with counts summed, not averaged, from the original UMI matrix
- DE testing must be run with edgeR's LRT; switching to single-cell-specific methods would break agreement with scPower's DE power estimates
- The significance threshold should be 0.05 with Bonferroni correction using the estimated number of expressed genes
- Simulation-based validation (muscat with edgeR downstream) showed DE power estimates in reasonable agreement with scPower's analytic values
- Expression filtering should keep roughly 67-81% of genes, consistent with cutoffs removing 19-33% of lowly expressed genes

## Limitations

- The pseudobulk approach requires sufficient samples per condition; data-driven priors in scPower only allow reliable overall power assessment for sample sizes roughly equal to or smaller than the pilot data sets
- Larger sample sizes may identify new significant DEGs with lower effect sizes that were not captured in pilot-study-based power computation
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than assumed

## Evidence

- [intro] the 'pseudobulk' approach in combination with classical differential gene expression methods such as edgeR44 and limma-voom45 outperforms single cell specific methods: "the “pseudobulk” approach in combination with classical differential gene expression methods such as edgeR44 and limma-voom45 outperforms single cell specific methods"
- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function 'create.pseudobulk': "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [discussion] best practice guidelines for differential gene expression with RNA-seq recommend cutoffs that remove between 19 and 33% of lowly expressed genes: "best practice guidelines for differential gene expression with RNA-seq recommend cutoffs that remove between 19 and 33% of lowly expressed genes"
- [readme] scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq: "scPower is a R package for design and power analysis of cell type specific interindividual DE and eQTL studies using single cell RNA-seq."
