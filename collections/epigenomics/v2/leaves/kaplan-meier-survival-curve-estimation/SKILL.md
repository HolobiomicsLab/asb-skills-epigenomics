---
name: kaplan-meier-survival-curve-estimation
description: Use when you have cancer patient clinical survival data (time, vital
  status) plus gene expression (e.g., TCGA expression for an eccDNA gene like EGFR)
  and need to test whether expression level stratification predicts prognosis.
when_to_use_negative:
- Input is raw WGS data needing eccDNA detection — use the AmpliconArchitect pipeline
  instead of survival analysis
- The goal is multivariate prognostic modeling with covariates (e.g., Cox regression),
  not two-group KM stratification by expression
- Clinical survival time or vital status is missing for samples, so no survival outcome
  can be modeled
tools:
- name: lifelines
  role: Fits KaplanMeierFitter curves per expression-stratified group and computes
    the log-rank test p-value
- name: pandas
  role: Loads, merges, and cleans TCGA expression and clinical tables on the sample
    barcode
- name: AmpliconArchitect
  role: 'Upstream: identifies the eccDNA elements (e.g., hsa_Chr7_3S_1) whose genes
    supply the expression-prognosis candidates'
- name: eccDNAdb
  role: Source of eccDNA profiles integrated with gene expression and clinical information
    for prognosis analysis
  repo: github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:fd9c7e68d92d932404c4941841ff3a9a580e50dbae9878fd2e25f56031547769
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - eccDNAdb
  - lifelines
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# kaplan-meier-survival-curve-estimation

## Summary

Estimates Kaplan-Meier overall-survival curves for patient groups stratified by a gene's expression level (e.g., EGFR in TCGA-LGG) and tests group differences with a lifelines log-rank test, linking eccDNA-carried genes to cancer prognosis as done in eccDNAdb.

## When to use

Use when you have cancer patient clinical survival data (time, vital status) plus gene expression (e.g., TCGA expression for an eccDNA gene like EGFR) and need to test whether expression level stratification predicts prognosis.

## When NOT to use

- Input is raw WGS data needing eccDNA detection — use the AmpliconArchitect pipeline instead of survival analysis
- The goal is multivariate prognostic modeling with covariates (e.g., Cox regression), not two-group KM stratification by expression
- Clinical survival time or vital status is missing for samples, so no survival outcome can be modeled

## Inputs

- TCGA-LGG clinical table: overall survival time and vital status per TCGA sample barcode
- Gene expression table (e.g., EGFR expression) keyed by TCGA sample barcode
- Cutoff rule for stratification (mean or median expression value)

## Outputs

- Survival curve figure with high/low-expression Kaplan-Meier curves and group labels (e.g., egfr_lgg_survival_curve.png)
- Results table/file with log-rank statistic, p-value, and per-group sample sizes (e.g., survival_results)

## How to apply

Load TCGA clinical information (overall survival time and vital status) and gene expression data (e.g., EGFR in TCGA-LGG) into pandas DataFrames, merge on the TCGA sample barcode, and drop samples missing survival time, status, or expression. Stratify patients into high- and low-expression groups using the mean/median expression value as the cutoff. Fit a lifelines KaplanMeierFitter per group, plot both curves on one figure with at-risk group labels, and compute the log-rank p-value with lifelines logrank_test. Save the curve figure (e.g., egfr_lgg_survival_curve.png) and write the log-rank statistic, p-value, and group sample sizes to a results file; a significant result (p < 0.05) with worse survival in the high-expression group reproduces the paper's Fig. 4D finding that high EGFR expression indicates poor LGG prognosis.

## Related tools

- **lifelines** (Fits KaplanMeierFitter curves per expression-stratified group and computes the log-rank test p-value)
- **pandas** (Loads, merges, and cleans TCGA expression and clinical tables on the sample barcode)
- **AmpliconArchitect** (Upstream: identifies the eccDNA elements (e.g., hsa_Chr7_3S_1) whose genes supply the expression-prognosis candidates)
- **eccDNAdb** (Source of eccDNA profiles integrated with gene expression and clinical information for prognosis analysis) — github.com/solo7773/eccDNAdb

## Examples

```
from lifelines import KaplanMeierFitter; from lifelines.statistics import logrank_test; kmf = KaplanMeierFitter(); kmf.fit(T, E); logrank_test(T_hi, T_lo, E_hi, E_lo)
```

## Evaluation signals

- Log-rank test yields p < 0.05 comparing high- vs low-expression groups
- High-EGFR group shows worse overall survival (lower KM curve) than the low-EGFR group, consistent with the paper's Fig. 4D
- Merged table contains survival time, vital status, and expression for every analyzed sample (no missing values after cleaning)
- Group sample sizes, log-rank statistic, and p-value are all reported in the results output
- Figure shows both survival curves with at-risk group labels and is saved as the expected PNG artifact

## Limitations

- All eccDNAs in the source database were identified by the AmpliconArchitect pipeline only, limiting the set of genes that can be tested for prognosis
- Dichotomizing by a single mean/median expression cutoff discards expression-level granularity and may be sensitive to cutoff choice
- More genetic and epigenetic information related to eccDNAs is not yet integrated, so confounders may not be accounted for in the KM comparison

## Evidence

- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE: "Gene expression and clinical data were downloaded from TCGA and CCLE"
- [abstract] eccDNAdb integrates expression and clinical data for clinical relevance: "With the integration of gene expression, clinical information and chromatin accessibility data"
- [discussion] EccDNA genes annotated with TCGA/CCLE expression data: "EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases."
- [readme] eccDNAdb repo README describes source code for eccDNA identification and gene annotation: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
