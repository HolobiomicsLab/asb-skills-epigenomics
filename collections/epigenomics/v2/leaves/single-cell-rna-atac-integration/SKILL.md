---
name: single-cell-rna-atac-integration
description: Use when matched scRNA-seq and scATAC-seq data are available (e.g. 10X_PBMC_15K
  PBMC) and candidate distal regulatory elements must be linked to their putative
  target genes and validated against cis-eQTLs.
when_to_use_negative:
- No matched scRNA-seq data are available for the scATAC-seq sample, so imputed gene
  expression cannot be fitted
- The goal is cell-type clustering or dimensionality reduction rather than linking
  distal elements to putative target genes
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: SnapATAC
  role: Imputes gene expression from accessibility, binarizes distal enhancer accessibility,
    and fits logistic-regression gene-peak pair associations via predictGenePeakPair
  repo: https://github.com/r3fang/SnapATAC
- name: R
  role: Runtime environment for SnapATAC (>= 3.4.0 & < 3.6.0, since 3.6 breaks the
    rhdf5 package)
- name: SnapTools
  role: Preprocessing module that builds the snap file containing cell-by-peak and
    cell-by-gene matrices used as input
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:f526e72184d940c3a2d3a938783e493ae1b7a2affd44a4caca77116448f58813
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-rna-atac-integration

## Summary

Integrates matched scRNA-seq expression data with scATAC-seq chromatin accessibility in SnapATAC by imputing gene expression from accessibility and binarizing distal enhancer accessibility, then fitting logistic-regression associations between imputed gene expression and linked distal peaks to identify enhancer–gene pairs.

## When to use

Use when matched scRNA-seq and scATAC-seq data are available (e.g. 10X_PBMC_15K PBMC) and candidate distal regulatory elements must be linked to their putative target genes and validated against cis-eQTLs.

## When NOT to use

- No matched scRNA-seq data are available for the scATAC-seq sample, so imputed gene expression cannot be fitted
- The goal is cell-type clustering or dimensionality reduction rather than linking distal elements to putative target genes

## Inputs

- scATAC-seq snap object (e.g. 10X_PBMC_15K PBMC scATAC-seq)
- Matched PBMC scRNA-seq gene expression matrix
- Gene names and gene loci (gene coordinates for the target gene set)
- Positive reference set (e.g. Fairfax monocyte interferon-γ/LPS cis-eQTLs) for benchmarking

## Outputs

- Ranked gene-peak pair predictions (gene_peak_pair_predictions.csv)
- Benchmark metrics against cis-eQTL reference (AUROC = 0.66, AUPRC = 0.68)

## How to apply

In SnapATAC (R, >= 3.4.0 & < 3.6.0), load the scATAC snap object (e.g. 10X_PBMC_15K) and the matched PBMC scRNA-seq expression data. Impute gene expression from chromatin accessibility and binarize distal enhancer accessibility, restricting analysis to a defined gene set (e.g. the 3000 PBMC differentially expressed genes). Call predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par, num.cores) to fit, per gene locus, a logistic-regression association between imputed gene expression and the accessibility of linked distal peaks, using multiple cores. Rank the resulting gene-peak pairs and benchmark them against an independent positive reference such as the Fairfax monocyte IFN-γ/LPS cis-eQTL set.

## Related tools

- **SnapATAC** (Imputes gene expression from accessibility, binarizes distal enhancer accessibility, and fits logistic-regression gene-peak pair associations via predictGenePeakPair) — https://github.com/r3fang/SnapATAC
- **R** (Runtime environment for SnapATAC (>= 3.4.0 & < 3.6.0, since 3.6 breaks the rhdf5 package))
- **SnapTools** (Preprocessing module that builds the snap file containing cell-by-peak and cell-by-gene matrices used as input) — https://github.com/r3fang/SnapTools

## Examples

```
predictions <- predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par=TRUE, num.cores=8)
```

## Evaluation signals

- Benchmark ranked gene-peak pairs against Fairfax monocyte IFN-γ/LPS cis-eQTL positives; expect AUROC ≈ 0.66 and AUPRC ≈ 0.68
- Gene-peak predictions are written as gene_peak_pair_predictions.csv with ranked pairs per gene locus
- Gene set restricted to the 3000 PBMC differentially expressed genes; check the input.mat matches this gene universe
- Confirms each gene's linked peaks show significant logistic-regression association with imputed gene expression rather than random linkage

## Limitations

- Prediction power is moderate (AUROC = 0.66, AUPRC = 0.68), so enhancer–gene links are putative and require independent validation such as cis-eQTLs
- R 3.6 does not work for the rhdf5 package, constraining the SnapATAC (R) runtime to R >= 3.4.0 & < 3.6.0
- Original SnapATAC is superseded by SnapATAC2 (https://github.com/kaizhang/SnapATAC2) for newer analyses

## Evidence

- [methods] SnapATAC links distal elements to putative target genes: "SnapATAC links distal elements to putative target genes"
- [methods] Integration of scRNA and scATAC data: "SnapATAC integrates scRNA and scATAC"
- [results] Enhancer-gene predictions match monocyte cis-eQTLs with moderate power: [evidence span withheld — anchor status: not_found]
- [readme] predictGenePeakPair function signature: "SnapATAC (Latest Updates: 2019-09-19)"
- [readme] R version constraint for SnapATAC: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
