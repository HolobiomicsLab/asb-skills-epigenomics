---
name: cancer-cohort-expression-profiling
description: Use when you have a defined gene set (e.g. genes on an eccDNA like hsa_Chr7_3S_1)
  and expression matrices from tumor and matched normal samples, and need per-gene
  expression ranks and t-test p-values per cancer cohort.
when_to_use_negative:
- The analysis goal is eccDNA identification from WGS data (eccDNAdb's AmpliconArchitect
  pipeline), not expression-level comparison of already-annotated eccDNA genes
- No normal/matched control samples exist for the cohort, making a tumor-vs-normal
  t-test uninterpretable
- The input is already a differential expression results table rather than a raw per-sample
  expression matrix
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: eccDNAdb
  role: Source of eccDNA gene annotations and the reference Tumor-vs-Normal box-plot-with-t-test
    presentation this profiling reproduces; scripts for annotating eccDNA-derived
    genes are in its repo
  repo: github.com/solo7773/eccDNAdb
- name: TCGA (GDC portal)
  role: Provides tumor (and clinical) gene expression matrices for the cancer cohort
    being profiled
- name: CCLE
  role: Provides cancer cell line expression data used alongside TCGA to annotate
    and compare eccDNA gene expression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:f3bbe77080ff574d77b4a75e314d3077efc98d76fad65dd59f0549c166c9e28f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCLE
  - TCGA (GDC portal)
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cancer-cohort-expression-profiling

## Summary

Profiles gene expression of eccDNA-derived genes across tumor and normal cohorts (TCGA, CCLE) and quantifies per-gene tumor-vs-normal differences with two-tailed Student's t-tests, mirroring the Tumor-vs-Normal box plots with p-values presented in eccDNAdb.

## When to use

Use when you have a defined gene set (e.g. genes on an eccDNA like hsa_Chr7_3S_1) and expression matrices from tumor and matched normal samples, and need per-gene expression ranks and t-test p-values per cancer cohort.

## When NOT to use

- The analysis goal is eccDNA identification from WGS data (eccDNAdb's AmpliconArchitect pipeline), not expression-level comparison of already-annotated eccDNA genes
- No normal/matched control samples exist for the cohort, making a tumor-vs-normal t-test uninterpretable
- The input is already a differential expression results table rather than a raw per-sample expression matrix

## Inputs

- Gene expression matrix (genes x samples) for a TCGA cancer cohort (e.g. TCGA-GBM) downloaded from the GDC portal
- Gene expression matrix for matched CCLE cancer cell lines from the CCLE data portal
- eccDNA gene annotations from eccDNAdb for the eccDNA of interest (e.g. gene set on hsa_Chr7_3S_1)
- Sample class labels distinguishing tumor vs normal (or tumor vs cell line) samples

## Outputs

- expression_ttest_results.csv with one row per eccDNA gene per cohort containing expression values, expression ranks, t-statistics, and p-values
- Per-gene expression profiles supporting Tumor vs. Normal box plots with p-values annotated next to each gene name

## How to apply

Download gene expression matrices for the target cohorts (e.g. TCGA-GBM from the GDC portal; matched CCLE cell lines from the CCLE portal), following the eccDNAdb approach of annotating eccDNA genes with expression data from TCGA and CCLE. Load eccDNA gene annotations from eccDNAdb (1270 eccDNAs, 54,901 annotated eccDNA genes across 42 cancers) and extract the gene set overlapping the eccDNA of interest. Subset the TCGA and CCLE matrices to these genes, compute per-gene expression profiles and expression ranks across tumor and normal samples, then run a two-tailed Student's t-test per gene per cohort using scipy.stats.ttest_ind. Compile per-gene expression values, ranks, t-statistics, and p-values into one table (e.g. expression_ttest_results.csv), with one row per eccDNA gene per cohort, matching the eccDNAdb presentation where p-values are displayed next to each gene name in Tumor vs. Normal box plots.

## Related tools

- **eccDNAdb** (Source of eccDNA gene annotations and the reference Tumor-vs-Normal box-plot-with-t-test presentation this profiling reproduces; scripts for annotating eccDNA-derived genes are in its repo) — github.com/solo7773/eccDNAdb
- **TCGA (GDC portal)** (Provides tumor (and clinical) gene expression matrices for the cancer cohort being profiled)
- **CCLE** (Provides cancer cell line expression data used alongside TCGA to annotate and compare eccDNA gene expression)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(tumor_expr.loc[gene], normal_expr.loc[gene])
```

## Evaluation signals

- Output table contains exactly one row per eccDNA gene per cohort (e.g. every gene on hsa_Chr7_3S_1 has a TCGA-GBM row)
- All expression values in the table are finite (no NaNs) for the retained genes
- Every row has a computed p-value in [0, 1] and a finite t-statistic from the two-tailed Student's t-test
- The demonstrated hsa_Chr7_3S_1 / TCGA-GBM entries are populated rather than empty
- Expression ranks are consistent with the per-gene expression values within each cohort

## Limitations

- eccDNA gene annotations used here derive from eccDNAdb, in which all eccDNAs were identified by the AmpliconArchitect pipeline only; eccDNAs missed by that pipeline are absent
- Coverage is restricted to human cancers in the current eccDNAdb release (more species and diseases are planned for future updates)
- CCLE cell lines lack matched normal counterparts, so the tumor-vs-normal comparison is defined within TCGA cohorts; cell-line comparisons are against tumor expression distributions

## Evidence

- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
- [discussion] EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases.: "EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases."
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes.: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [results] eccDNAdb presents eccDNA gene expression in Tumor vs. Normal as box plots, with p-values from Student's t-test displayed next to each gene name: [evidence span withheld — anchor status: not_found]
