---
name: tumor-vs-normal-differential-expression-ttest
description: Use when a user asks whether eccDNA genes from eccDNAdb show differential
  expression between tumor and normal samples in TCGA cohorts (e.g. TCGA-GBM) or matched
  CCLE cell lines, and wants per-gene t-statistics and p-values.
when_to_use_negative:
- The input is RNA-seq/ATAC-seq data not linked to eccDNAdb eccDNA gene annotations
  — this skill is specific to eccDNA-derived gene sets
- Only a single sample group is available (no matched normal or comparator samples),
  making a tumor-vs-normal t-test undefined
- The goal is de novo eccDNA identification from WGS (use the AmpliconArchitect pipeline
  in eccDNAdb instead, not expression testing)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: eccDNAdb
  role: Source of eccDNA annotations and the eccDNA-derived gene sets to be tested,
    plus reference Tumor vs. Normal box plots with Student's t-test p-values
  repo: http://www.eccdnadb.org
- name: TCGA
  role: Provider of tumor and normal gene expression data for cancer cohorts such
    as TCGA-GBM, downloaded via the GDC portal
- name: CCLE
  role: Provider of cancer cell-line gene expression data used as an independent cohort
    for per-gene tumor (cell-line) expression comparison
- name: scipy.stats.ttest_ind
  role: Performs the two-tailed Student's t-test comparing tumor vs normal expression
    for each eccDNA gene
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:6a692e7c160f7ab7c2cd8fd2ad59a0fe6adf7dc2147a2f16ada17921f37b365e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCLE
  - TCGA
  - eccDNAdb
  - scipy.stats.ttest_ind
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tumor-vs-normal-differential-expression-ttest

## Summary

Quantifies expression differences between tumor and normal samples for eccDNA-derived genes by subsetting TCGA/CCLE expression matrices to eccDNA gene sets (e.g. hsa_Chr7_3S_1) and computing per-gene two-tailed Student's t-tests, as visualized in eccDNAdb's Tumor vs. Normal box plots with p-values (e.g. TCGA-GBM, Fig. 4C).

## When to use

Use when a user asks whether eccDNA genes from eccDNAdb show differential expression between tumor and normal samples in TCGA cohorts (e.g. TCGA-GBM) or matched CCLE cell lines, and wants per-gene t-statistics and p-values.

## When NOT to use

- The input is RNA-seq/ATAC-seq data not linked to eccDNAdb eccDNA gene annotations — this skill is specific to eccDNA-derived gene sets
- Only a single sample group is available (no matched normal or comparator samples), making a tumor-vs-normal t-test undefined
- The goal is de novo eccDNA identification from WGS (use the AmpliconArchitect pipeline in eccDNAdb instead, not expression testing)

## Inputs

- TCGA gene expression matrix (e.g. TCGA-GBM) downloaded from the GDC portal
- CCLE cell-line gene expression matrix from the CCLE data portal
- eccDNA gene annotations from eccDNAdb (e.g. genes on eccDNA hsa_Chr7_3S_1)
- Sample class labels distinguishing tumor vs normal samples per cohort

## Outputs

- expression_ttest_results.csv with one row per eccDNA gene per cohort containing expression values, expression ranks, t-statistics, and p-values
- Per-gene tumor vs normal box plots with Student's t-test p-values annotated next to gene names (analogous to eccDNAdb Fig. 4C)

## How to apply

Download gene expression matrices for the target cohort (e.g. TCGA-GBM from the GDC portal; CCLE cell-line expression from the CCLE data portal) — eccDNAdb itself integrated 'gene expression and clinical data ... from TCGA and CCLE'. Load eccDNA gene annotations from eccDNAdb and extract the gene set overlapping the eccDNA of interest (e.g. genes on hsa_Chr7_3S_1). Subset both expression matrices to these genes, compute per-gene expression profiles and expression ranks across tumor and normal samples, then run a two-tailed Student's t-test per gene per cohort using scipy.stats.ttest_ind, mirroring the p-values eccDNAdb displays next to each gene name in its Tumor vs. Normal box plots. Compile expression values, ranks, t-statistics, and p-values into one table written as expression_ttest_results.csv.

## Related tools

- **eccDNAdb** (Source of eccDNA annotations and the eccDNA-derived gene sets to be tested, plus reference Tumor vs. Normal box plots with Student's t-test p-values) — http://www.eccdnadb.org
- **TCGA** (Provider of tumor and normal gene expression data for cancer cohorts such as TCGA-GBM, downloaded via the GDC portal)
- **CCLE** (Provider of cancer cell-line gene expression data used as an independent cohort for per-gene tumor (cell-line) expression comparison)
- **scipy.stats.ttest_ind** (Performs the two-tailed Student's t-test comparing tumor vs normal expression for each eccDNA gene)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(tumor_expr[gene], normal_expr[gene], equal_var=True)  # per eccDNA gene, cohort=T CGA-GBM; results -> expression_ttest_results.csv
```

## Evaluation signals

- expression_ttest_results.csv contains exactly one row per eccDNA gene per cohort (e.g. all genes of hsa_Chr7_3S_1 in TCGA-GBM)
- All expression values in the output table are finite (no NaNs after subsetting to eccDNA genes)
- Every gene row has a computed two-tailed t-statistic and a p-value in [0, 1]
- The hsa_Chr7_3S_1 / TCGA-GBM entries demonstrated in eccDNAdb (Fig. 4C-style) are populated
- Per-gene box plots of tumor vs normal expression reproduce the ordering of p-values displayed next to gene names in eccDNAdb

## Limitations

- eccDNAdb's eccDNA calls were all identified by the AmpliconArchitect pipeline only, so the eccDNA gene sets tested are limited to that pipeline's detections
- eccDNAdb coverage is human cancers only; cohorts outside its 42 cancers / 480 samples may lack eccDNA gene annotations
- Student's t-test on expression does not model covariates or batch effects; eccDNAdb itself presents only box plots with t-test p-values, not adjusted models

## Evidence

- [results] eccDNAdb presents eccDNA gene expression in Tumor vs. Normal as box plots, with p-values from Student's t-test displayed next to each gene name (e.g., hsa_Chr7_3S_1 genes in TCGA-GBM, Fig. 4C).: "Gene expression and clinical data were downloaded from TCGA and CCLE"
- [discussion] EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases.: "EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases."
- [abstract] With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of eccDNAs in cancers.: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of"
- [discussion] Limitation: all eccDNAs in the database were identified by the AmpliconArchitect pipeline only: "will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
