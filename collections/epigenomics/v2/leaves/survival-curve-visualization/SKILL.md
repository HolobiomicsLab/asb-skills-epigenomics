---
name: survival-curve-visualization
description: Use when you have time-to-event clinical data (overall survival time
  + vital status) matched to gene expression, and need to test whether high vs low
  expression of an eccDNA gene is associated with prognosis.
when_to_use_negative:
- The cohort lacks follow-up time or censoring status, so time-to-event modeling is
  impossible and a simple expression-outcome association test would be needed instead
- The question is which eccDNAs exist in a tumor's WGS data (eccDNA identification)
  rather than testing clinical relevance of an already-annotated eccDNA gene
- You need chromatin accessibility or regulatory interpretation of eccDNA rather than
  a survival association
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: lifelines
  role: Fits Kaplan–Meier survival curves per expression group and computes the log-rank
    test p-value
- name: pandas
  role: Merges TCGA expression and clinical tables on the sample barcode and drops
    samples missing survival time, status, or expression
- name: AmpliconArchitect
  role: Upstream identification of the eccDNAs whose resident genes (e.g., EGFR on
    hsa_Chr7_3S_1) are tested for prognostic association
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:259a3878b7cd80ada6fe826d3553270cdebbc09b636d111cdc1d0f6180f7b1d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - lifelines
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# survival-curve-visualization

## Summary

Kaplan–Meier overall-survival analysis of patient cohorts stratified by a gene's expression level (e.g., EGFR in TCGA-LGG), used to test whether eccDNA-resident gene expression relates to cancer prognosis.

## When to use

Use when you have time-to-event clinical data (overall survival time + vital status) matched to gene expression, and need to test whether high vs low expression of an eccDNA gene is associated with prognosis.

## When NOT to use

- The cohort lacks follow-up time or censoring status, so time-to-event modeling is impossible and a simple expression-outcome association test would be needed instead
- The question is which eccDNAs exist in a tumor's WGS data (eccDNA identification) rather than testing clinical relevance of an already-annotated eccDNA gene
- You need chromatin accessibility or regulatory interpretation of eccDNA rather than a survival association

## Inputs

- TCGA clinical table with overall survival time and vital status keyed by TCGA sample barcode
- Gene expression table (e.g., EGFR expression for TCGA-LGG samples) keyed by TCGA sample barcode
- List of genes on the eccDNA of interest (e.g., genes of eccDNA hsa_Chr7_3S_1)

## Outputs

- Kaplan–Meier survival curve figure with high/low expression curves and group labels (e.g., egfr_lgg_survival_curve.png)
- Survival results record with log-rank statistic, p-value, and group sample sizes (e.g., survival_results)

## How to apply

Load TCGA clinical information (overall survival time, vital status) and gene expression (e.g., EGFR from TCGA-LGG) into pandas DataFrames, then merge on the TCGA sample barcode and drop samples missing survival time, status, or expression. Stratify patients into high- and low-expression groups using the mean/median expression value as the cutoff. Fit a Kaplan–Meier curve per group with lifelines KaplanMeierFitter and plot both curves on a single figure with at-risk group labels. Compute the log-rank test p-value with lifelines logrank_test and report the statistic, p-value, and per-group sample sizes; treatment as successful requires p < 0.05 with the high-expression group showing worse overall survival.

## Related tools

- **lifelines** (Fits Kaplan–Meier survival curves per expression group and computes the log-rank test p-value)
- **pandas** (Merges TCGA expression and clinical tables on the sample barcode and drops samples missing survival time, status, or expression)
- **AmpliconArchitect** (Upstream identification of the eccDNAs whose resident genes (e.g., EGFR on hsa_Chr7_3S_1) are tested for prognostic association)

## Examples

```
from lifelines.statistics import logrank_test; lr = logrank_test(T_high, T_low, E_high, E_low); print(lr.test_statistic, lr.p_value)
```

## Evaluation signals

- Log-rank test yields p < 0.05 comparing high vs low expression groups
- High-EGFR group shows worse overall survival than the low-EGFR group in TCGA-LGG, consistent with the paper's Fig. 4D
- Merged cohort excludes samples missing survival time, vital status, or EGFR expression before fitting curves
- Figure contains two Kaplan–Meier curves with at-risk group labels; results record includes log-rank statistic, p-value, and both group sample sizes

## Limitations

- Choice of stratification cutoff (mean vs median expression) changes group membership and can alter the log-rank result
- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so eccDNA gene sets analyzed this way inherit that detection limitation
- Few sequencing datasets are available for alternative eccDNA techniques such as CIRCLE-seq and CIDER-Seq, limiting cross-validation of the eccDNA gene definitions

## Evidence

- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers: "Gene expression and clinical data were downloaded from TCGA and CCLE and then used to investigate the roles of eccDNAs in cancers"
- [abstract] Integration of expression, clinical, and accessibility data: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of"
- [readme] eccDNAdb source code purpose: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
