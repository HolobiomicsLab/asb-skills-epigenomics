---
name: ccle-expression-data-retrieval
description: Use when a task needs expression values for cancer cell lines (CCLE)
  or TCGA tumor/normal cohorts to quantify differential expression of genes located
  on specific eccDNAs, e.g. genes on hsa_Chr7_3S_1 in TCGA-GBM.
when_to_use_negative:
- The eccDNAs of interest are from non-human species or non-cancer diseases, since
  eccDNAdb currently covers human cancers only
- The task requires eccDNA detection from raw WGS data (use AmpliconArchitect) rather
  than retrieval/analysis of already-annotated eccDNA gene expression
- Only normal tissue expression is needed and no tumor-vs-normal comparison is planned
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: eccDNAdb
  role: Source of eccDNA gene annotations (54,901 eccDNA genes) and precomputed gene
    expression from TCGA and CCLE
  repo: github.com/solo7773/eccDNAdb
- name: CCLE
  role: Cancer cell line gene expression data downloaded to compare/annotate eccDNA
    gene expression
- name: TCGA (GDC portal)
  role: Patient tumor and normal gene expression and clinical data for the tumor-vs-normal
    t-tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:4c46384ebfb5328d1371a4668fd34cad876a254c76979c2f36c67db1a94b9fce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCLE
  - TCGA (GDC portal)
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ccle-expression-data-retrieval

## Summary

Retrieve and quantify cancer gene expression profiles from the CCLE (and matched TCGA cohorts) so that eccDNA-derived genes can be compared between tumor and normal samples using Student's t-tests, mirroring the eccDNAdb approach for eccDNA gene expression annotation.

## When to use

Use when a task needs expression values for cancer cell lines (CCLE) or TCGA tumor/normal cohorts to quantify differential expression of genes located on specific eccDNAs, e.g. genes on hsa_Chr7_3S_1 in TCGA-GBM.

## When NOT to use

- The eccDNAs of interest are from non-human species or non-cancer diseases, since eccDNAdb currently covers human cancers only
- The task requires eccDNA detection from raw WGS data (use AmpliconArchitect) rather than retrieval/analysis of already-annotated eccDNA gene expression
- Only normal tissue expression is needed and no tumor-vs-normal comparison is planned

## Inputs

- CCLE gene expression matrix (downloaded from the CCLE data portal)
- TCGA cohort gene expression matrix (e.g. TCGA-GBM, from the GDC portal)
- eccDNA gene annotations from eccDNAdb (genes overlapping a target eccDNA such as hsa_Chr7_3S_1)

## Outputs

- expression_ttest_results.csv: one row per eccDNA gene per cohort with expression values, ranks, t-statistics and two-tailed Student's t-test p-values
- Per-gene tumor vs normal expression profiles and ranks for the eccDNA gene set

## How to apply

Download the gene expression matrices from the CCLE data portal and, for patient cohorts, the TCGA GDC portal (eccDNAdb itself downloads gene expression and clinical data from TCGA and CCLE). Load eccDNA gene annotations from eccDNAdb (1270 eccDNAs across 480 samples of 42 cancers) and extract the gene set overlapping the eccDNA of interest. Subset the expression matrices to these eccDNA genes, compute per-gene expression profiles and ranks across tumor vs normal samples, then run a two-tailed Student's t-test per gene (scipy.stats.ttest_ind) within each cancer cohort, compiling values, t-statistics and p-values into expression_ttest_results.csv. This reproduces how eccDNAdb presents eccDNA gene expression as Tumor vs. Normal box plots with per-gene p-values (e.g. Fig. 4C for hsa_Chr7_3S_1 genes in TCGA-GBM).

## Related tools

- **eccDNAdb** (Source of eccDNA gene annotations (54,901 eccDNA genes) and precomputed gene expression from TCGA and CCLE) — github.com/solo7773/eccDNAdb
- **CCLE** (Cancer cell line gene expression data downloaded to compare/annotate eccDNA gene expression)
- **TCGA (GDC portal)** (Patient tumor and normal gene expression and clinical data for the tumor-vs-normal t-tests)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(tumor_expr[gene], normal_expr[gene], equal_var=True)  # per eccDNA gene, e.g. hsa_Chr7_3S_1 genes in TCGA-GBM
```

## Evaluation signals

- Output table contains one row per eccDNA gene per cohort
- All rows have finite expression values and computed (non-NaN) p-values
- The demonstrated hsa_Chr7_3S_1 / TCGA-GBM entries are populated
- t-statistics and p-values match a two-tailed scipy.stats.ttest_ind rerun on the same subsets

## Limitations

- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so eccDNA gene sets carry that pipeline's sensitivity limitations
- Currently human cancers only; more species and diseases are planned for future updates
- Not all genetic and epigenetic information related to eccDNAs is yet integrated into the database

## Evidence

- [results] eccDNAdb presents eccDNA gene expression in Tumor vs. Normal as box plots, with p-values from Student's t-test displayed next to each gene name: [evidence span withheld — anchor status: not_found]
- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
- [readme] The number notation at the beginning of a script filename indicates the order of an analysis. It is easy to follow the steps to repeat the analyses in our study.: "The number notation at the beginning of a script filename indicates the order of an analysis. It is easy to follow the steps to repeat the analyses in our study."
- [abstract] A total number of 54,901 eccDNA genes were annotated and included in the database as well: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
