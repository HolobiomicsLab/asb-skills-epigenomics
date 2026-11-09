---
name: tcga-gdc-expression-data-retrieval
description: Use when an analysis needs tumor/normal gene expression values for specific
  genes (e.g., genes on an eccDNA like hsa_Chr7_3S_1) in a cancer cohort such as TCGA-GBM
  or CCLE cell lines, to compute expression differences and per-gene p-values.
when_to_use_negative:
- The eccDNAs or genes of interest lie outside the cohorts covered by eccDNAdb (e.g.,
  non-human species or non-cancer diseases, which the database does not yet include)
- The goal is eccDNA identification from WGS data rather than expression analysis
  of already-annotated eccDNA genes — that requires the AmpliconArchitect pipeline,
  not expression matrices
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: TCGA
  role: Source of tumor gene expression (and clinical) data downloaded via the GDC
    portal for tumor-vs-normal expression comparison of eccDNA genes
- name: CCLE
  role: Source of cancer cell line gene expression data used alongside TCGA to annotate
    eccDNA genes with expression
- name: eccDNAdb
  role: Provides the eccDNA gene annotations (54,901 annotated eccDNA genes) used
    to define which genes are subset from the expression matrices
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:1091ad61e145e55e95631b385ba016003de38749f2e811d9041c7201d31af9cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - CCLE
  - TCGA
  - TCGA (GDC portal)
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tcga-gdc-expression-data-retrieval

## Summary

Retrieves gene expression matrices for cancer cohorts from TCGA (GDC portal) and CCLE so that eccDNA gene expression can be compared between tumor and normal samples, as done in eccDNAdb where per-gene expression is presented as Tumor vs. Normal box plots with Student's t-test p-values.

## When to use

Use when an analysis needs tumor/normal gene expression values for specific genes (e.g., genes on an eccDNA like hsa_Chr7_3S_1) in a cancer cohort such as TCGA-GBM or CCLE cell lines, to compute expression differences and per-gene p-values.

## When NOT to use

- The eccDNAs or genes of interest lie outside the cohorts covered by eccDNAdb (e.g., non-human species or non-cancer diseases, which the database does not yet include)
- The goal is eccDNA identification from WGS data rather than expression analysis of already-annotated eccDNA genes — that requires the AmpliconArchitect pipeline, not expression matrices

## Inputs

- TCGA (GDC portal) gene expression matrix for a cancer cohort (e.g., TCGA-GBM tumor and matched normal samples)
- CCLE data portal gene expression matrix for matched cancer cell lines
- eccDNA gene annotations from eccDNAdb identifying the gene set overlapping an eccDNA of interest (e.g., hsa_Chr7_3S_1)

## Outputs

- expression_ttest_results.csv with one row per eccDNA gene per cohort, containing per-gene expression values, expression ranks, t-statistics, and two-tailed Student's t-test p-values
- Per-gene tumor vs. normal expression profiles suitable for box-plot display as in eccDNAdb Fig. 4C

## How to apply

Download the gene expression matrices for the target cohorts (e.g., TCGA-GBM and matched CCLE cell lines) from the TCGA GDC portal and the CCLE data portal, as eccDNAdb did when it integrated gene expression from both sources. Load eccDNA gene annotations from eccDNAdb and extract the gene set overlapping the eccDNA of interest (e.g., genes on hsa_Chr7_3S_1). Subset the expression matrices to these eccDNA genes, computing per-gene expression profiles and expression ranks across tumor and normal samples. For each gene, perform a two-tailed Student's t-test comparing tumor vs. normal expression within each cohort (scipy.stats.ttest_ind), mirroring the p-values eccDNAdb displays next to each gene name in its box plots. Compile per-gene expression values, ranks, t-statistics, and p-values into a single table (e.g., expression_ttest_results.csv) with one row per eccDNA gene per cohort.

## Related tools

- **TCGA** (Source of tumor gene expression (and clinical) data downloaded via the GDC portal for tumor-vs-normal expression comparison of eccDNA genes)
- **CCLE** (Source of cancer cell line gene expression data used alongside TCGA to annotate eccDNA genes with expression)
- **eccDNAdb** (Provides the eccDNA gene annotations (54,901 annotated eccDNA genes) used to define which genes are subset from the expression matrices) — https://github.com/solo7773/eccDNAdb

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(tumor_expr[gene], normal_expr[gene], equal_var=True)  # two-tailed Student's t-test per eccDNA gene
```

## Evaluation signals

- The output table contains one row per eccDNA gene per cohort, with finite expression values and computed p-values
- The demonstrated hsa_Chr7_3S_1 entries for TCGA-GBM are populated in the results table
- Each gene row carries a t-statistic and a two-tailed Student's t-test p-value consistent with the tumor vs. normal comparison
- Expression ranks are computed across tumor and normal samples within each cohort and are finite/ordered

## Limitations

- Matched normal samples may be limited for some TCGA cohorts, and CCLE provides cell line rather than matched-normal expression, constraining tumor-vs-normal comparisons
- eccDNAdb content reflects only eccDNAs identified by the AmpliconArchitect pipeline plus literature curation, so the eccDNA gene annotations available for subsetting are bounded by that coverage
- The database currently covers human cancers only (42 cancers, 1270 eccDNAs in 480 samples at publication); other species and diseases are not supported

## Evidence

- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
- [results] eccDNAdb presents eccDNA gene expression in Tumor vs. Normal as box plots, with p-values from Student's t-test: [evidence span withheld — anchor status: not_found]
- [abstract] A total number of 54,901 eccDNA genes were annotated and included in the database as well: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [abstract] With the integration of gene expression, clinical information and chromatin accessibility data: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of"
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
