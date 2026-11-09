---
name: log-rank-significance-testing
description: Use when two patient groups (e.g., high vs low EGFR expression in TCGA-LGG)
  have overall survival time and vital status, and you must test survival differences
  with a log-rank test.
when_to_use_negative:
- Clinical table lacks survival time or vital status fields, so the merged survival
  dataset would be empty after dropping missing samples.
- The analysis goal is detecting eccDNA elements from WGS data with AmpliconArchitect
  rather than testing expression–prognosis association.
- More than two stratification groups are planned, since the workflow as specified
  fits exactly two Kaplan–Meier curves and one pairwise logrank_test.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_2640
tools:
- name: lifelines
  role: Fits per-group KaplanMeierFitter survival curves and computes the log-rank
    statistic and p-value via logrank_test
- name: pandas
  role: Loads, merges on the TCGA sample barcode, and filters the clinical and expression
    tables before stratification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:7ab441f51b6e35f84ae02754a6539ecfb0cdbb12b6b9db5dedf5d93ad2d8fd0a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - lifelines
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# log-rank-significance-testing

## Summary

A Kaplan–Meier survival analysis skill that stratifies cancer patients by a gene's expression level (e.g., EGFR in TCGA-LGG) using a mean/median cutoff, fits per-group KM curves with lifelines, and computes a log-rank p-value to test whether high expression is associated with prognosis.

## When to use

Use when two patient groups (e.g., high vs low EGFR expression in TCGA-LGG) have overall survival time and vital status, and you must test survival differences with a log-rank test.

## When NOT to use

- Clinical table lacks survival time or vital status fields, so the merged survival dataset would be empty after dropping missing samples.
- The analysis goal is detecting eccDNA elements from WGS data with AmpliconArchitect rather than testing expression–prognosis association.
- More than two stratification groups are planned, since the workflow as specified fits exactly two Kaplan–Meier curves and one pairwise logrank_test.

## Inputs

- TCGA clinical table with overall survival time and vital status per sample barcode
- Gene expression matrix for the eccDNA-derived gene of interest (e.g., EGFR expression in TCGA-LGG)
- TCGA sample barcodes shared between the clinical and expression tables

## Outputs

- Kaplan–Meier survival curve figure comparing high- vs low-expression groups (e.g., egfr_lgg_survival_curve.png) with at-risk group labels
- Log-rank test statistic, p-value, and per-group sample sizes written to a results file (e.g., survival_results)

## How to apply

Load the clinical table (overall survival time and vital status) and gene expression data from TCGA into pandas DataFrames, then merge them on the TCGA sample barcode and drop samples missing survival time, status, or expression. Stratify patients into high- and low-expression groups using the mean/median expression value as the cutoff. Fit a KaplanMeierFitter per group and plot both curves on a single figure with at-risk group labels, then compute the log-rank statistic and p-value with lifelines logrank_test. Write the log-rank statistic, p-value, and group sample sizes to a results file (e.g., survival_results) and save the curve figure (e.g., egfr_lgg_survival_curve.png). The task succeeds if the log-rank test yields p < 0.05 with the high-expression group showing worse overall survival, as in the paper's Fig. 4D finding for EGFR in TCGA-LGG.

## Related tools

- **lifelines** (Fits per-group KaplanMeierFitter survival curves and computes the log-rank statistic and p-value via logrank_test)
- **pandas** (Loads, merges on the TCGA sample barcode, and filters the clinical and expression tables before stratification)

## Examples

```
from lifelines.statistics import logrank_test; lr = logrank_test(T_high, T_low, E_high, E_low); print(lr.test_statistic, lr.p_value)
```

## Evaluation signals

- Log-rank test returns p < 0.05 with the high-expression group showing worse overall survival (the expected direction for EGFR in TCGA-LGG per Fig. 4D).
- Merged table contains no samples missing survival time, vital status, or EGFR expression after filtering.
- Group sample sizes for high- and low-EGFR strata are reported and sum to the number of retained samples.
- Results file contains the log-rank statistic, p-value, and both group sample sizes; survival curve figure exists with both groups plotted on one axis.
- Survival curves are stratified by the mean/median EGFR expression cutoff as specified, not an arbitrary percentile.

## Limitations

- Stratification depends on the cutoff choice (mean vs median expression), which is a decision point that can change the log-rank result and its significance.
- The workflow was validated for overall survival in a single cancer type (TCGA-LGG with EGFR); extrapolating to other genes, endpoints, or cohorts is untested here.
- Clinical data quality is bounded by what TCGA provides; samples missing survival time, status, or expression are discarded, which can reduce power.

## Evidence

- [other] Stratify patients into high- and low-EGFR groups using the mean/median EGFR expression value as the cutoff.: [evidence span withheld — anchor status: not_found]
- [other] Compute the log-rank test p-value comparing the two groups with lifelines logrank_test.: [evidence span withheld — anchor status: not_found]
- [other] high expression of EGFR indicates poor prognosis of low-grade glioma (TCGA-LGG): [evidence span withheld — anchor status: not_found]
- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
