---
name: tcga-clinical-expression-data-integration
description: Use when TCGA gene expression (e.g., EGFR in TCGA-LGG) must be merged
  with overall-survival clinical metadata and compared between expression-defined
  patient groups to assess prognosis.
when_to_use_negative:
- No clinical survival time or vital-status metadata is available for the cohort,
  so no survival comparison can be performed
- The question concerns eccDNA discovery/identification itself (AmpliconArchitect
  on WGS data) rather than downstream prognosis testing — the article notes all eccDNAs
  in eccDNAdb were identified by the AmpliconArchitect pipeline only, which is a separate
  workflow
- The outcome is not overall survival (e.g., disease-free survival or response data),
  since this workflow is defined specifically around OS time and vital status
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Load, merge (on TCGA sample barcode), and clean expression and clinical tables;
    stratify expression groups by mean/median cutoff
- name: lifelines
  role: KaplanMeierFitter fits per-group survival curves; logrank_test computes the
    p-value comparing high- vs low-EGFR groups
- name: TCGA
  role: Source of the gene expression and clinical (overall survival, vital status)
    data merged in this workflow
- name: eccDNAdb
  role: Reference implementation/scripts whose workflow these survival analyses reproduce;
    eccDNA genes like EGFR on hsa_Chr7_3S_1 are selected from its annotations
  repo: github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:5937fc10e36ed4fd7fe14a74adc21765495658abb7d9767fe5878290ae469b4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - TCGA
  - eccDNAdb
  - lifelines
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tcga-clinical-expression-data-integration

## Summary

Links TCGA gene expression values with clinical survival metadata to test whether eccDNA-derived gene expression (e.g., EGFR from eccDNA hsa_Chr7_3S_1 in TCGA-LGG) predicts patient prognosis. This mirrors how eccDNAdb integrates expression and clinical data to determine clinical relevance of eccDNAs in cancers.

## When to use

Use when TCGA gene expression (e.g., EGFR in TCGA-LGG) must be merged with overall-survival clinical metadata and compared between expression-defined patient groups to assess prognosis.

## When NOT to use

- No clinical survival time or vital-status metadata is available for the cohort, so no survival comparison can be performed
- The question concerns eccDNA discovery/identification itself (AmpliconArchitect on WGS data) rather than downstream prognosis testing — the article notes all eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, which is a separate workflow
- The outcome is not overall survival (e.g., disease-free survival or response data), since this workflow is defined specifically around OS time and vital status

## Inputs

- TCGA-LGG clinical table: overall survival time and vital status keyed by TCGA sample barcode
- TCGA gene expression matrix (e.g., EGFR expression values) keyed by TCGA sample barcode
- Cutoff definition for dichotomizing expression (mean or median)

## Outputs

- Kaplan–Meier survival curve figure comparing high- vs low-expression groups (saved as egfr_lgg_survival_curve.png in the source workflow)
- Log-rank test statistic, p-value, and per-group sample sizes written to a results file (survival_results in the source workflow)

## How to apply

Load the TCGA overall-survival time and vital-status clinical table and the gene expression table (e.g., EGFR for TCGA-LGG) into pandas DataFrames, then merge them on the TCGA sample barcode and drop samples missing survival time, status, or expression. Stratify patients into high- and low-expression groups using the mean or median expression value as the cutoff — document which cutoff is chosen since it changes group sizes. Fit a Kaplan–Meier curve per group with lifelines KaplanMeierFitter and overlay both curves with group labels, then compute the log-rank p-value with lifelines logrank_test. Validate by checking that the log-rank test yields p < 0.05 with the high-expression group showing worse overall survival, as reported for high EGFR expression in low-grade glioma (paper Fig. 4D).

## Related tools

- **pandas** (Load, merge (on TCGA sample barcode), and clean expression and clinical tables; stratify expression groups by mean/median cutoff)
- **lifelines** (KaplanMeierFitter fits per-group survival curves; logrank_test computes the p-value comparing high- vs low-EGFR groups)
- **TCGA** (Source of the gene expression and clinical (overall survival, vital status) data merged in this workflow)
- **eccDNAdb** (Reference implementation/scripts whose workflow these survival analyses reproduce; eccDNA genes like EGFR on hsa_Chr7_3S_1 are selected from its annotations) — github.com/solo7773/eccDNAdb

## Examples

```
from lifelines.statistics import logrank_test; lr = logrank_test(durations_high, durations_low, event_observed_high=events_high, event_observed_low=events_low); print(lr.p_value)
```

## Evaluation signals

- After merging on the TCGA sample barcode and dropping NaNs, no samples retain missing survival time, status, or EGFR expression values
- High- and low-expression group sample sizes sum to the merged cohort size and are reported in the output
- Log-rank test yields p < 0.05 with the high-EGFR group showing worse overall survival, matching the paper's Fig. 4D finding for TCGA-LGG
- KM figure shows two labeled curves (high/low EGFR) with the high-expression curve consistently below the low-expression curve
- Output file (survival_results) contains the log-rank statistic, p-value, and both group sample sizes

## Limitations

- All eccDNAs (and hence eccDNA genes) analyzed come from the AmpliconArchitect pipeline only, per the article's stated limitation, so gene sets missed by that pipeline are absent
- Dichotomizing expression by a mean/median cutoff is arbitrary; results (p-value, group sizes) depend on the chosen cutoff, and no multivariate adjustment for confounders is part of this workflow
- More species and diseases were not yet covered (human cancers only), and more genetic/epigenetic information was not yet integrated in eccDNAdb

## Evidence

- [abstract] Gene expression and clinical data downloaded from TCGA drive survival testing: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
- [abstract] Integration rationale for linking expression with clinical information: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of eccDNAs in"
- [readme] Repository provides source scripts for the annotation workflow underlying these analyses: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
