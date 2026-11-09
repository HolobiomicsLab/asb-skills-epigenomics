---
name: gene-expression-median-stratification
description: Use when you have per-sample gene expression values (e.g., TCGA RNA data)
  matched to clinical survival metadata and need binary high/low expression groups
  for log-rank/KM prognosis testing of an eccDNA gene.
when_to_use_negative:
- The analysis goal is differential expression between tumor and normal samples rather
  than survival stratification by expression level
- Clinical survival time or vital status is unavailable for the cohort, since merged
  samples lacking these fields are dropped and the stratification cannot support a
  log-rank test
- The gene of interest is not among the eccDNA genes of the eccDNA element under study
  (e.g., not in the gene set of eccDNA hsa_Chr7_3S_1), so the eccDNA–prognosis question
  does not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Load the TCGA-LGG expression and clinical tables, merge on sample barcode,
    drop incomplete rows, and compute the mean/median expression cutoff for stratification
- name: lifelines
  role: Consumes the high/low strata produced by this skill for Kaplan–Meier curve
    fitting (KaplanMeierFitter) and log-rank testing (logrank_test)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:15a074e56a0b96cc410ea23e4711c2783c2f181f2ca1d0f8198f63e48822d5f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - lifelines
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-expression-median-stratification

## Summary

Stratify cancer patients into high- vs low-expression groups for a gene of interest (e.g., EGFR in TCGA-LGG) using the mean/median expression value as the cutoff, as a prerequisite for Kaplan–Meier survival analysis of eccDNA-derived genes.

## When to use

Use when you have per-sample gene expression values (e.g., TCGA RNA data) matched to clinical survival metadata and need binary high/low expression groups for log-rank/KM prognosis testing of an eccDNA gene.

## When NOT to use

- The analysis goal is differential expression between tumor and normal samples rather than survival stratification by expression level
- Clinical survival time or vital status is unavailable for the cohort, since merged samples lacking these fields are dropped and the stratification cannot support a log-rank test
- The gene of interest is not among the eccDNA genes of the eccDNA element under study (e.g., not in the gene set of eccDNA hsa_Chr7_3S_1), so the eccDNA–prognosis question does not apply

## Inputs

- Gene expression matrix for the cancer cohort (e.g., EGFR expression values per TCGA-LGG sample, downloaded from TCGA)
- Clinical metadata table with overall survival time and vital status keyed by TCGA sample barcode

## Outputs

- Patient-level table with sample barcode, expression value, high/low stratum label, survival time, and vital status
- Group sample sizes for the high- and low-expression strata (to be written to survival_results alongside the log-rank statistic and p-value)

## How to apply

Load the gene expression table (e.g., EGFR expression from TCGA) and the clinical table (overall survival time and vital status) into pandas DataFrames, then merge them on the TCGA sample barcode and drop samples missing survival time, vital status, or expression. Compute the mean/median expression value across the merged cohort and use it as the cutoff to assign each patient to a high- or low-expression group. This binarization feeds directly into per-group Kaplan–Meier fitting with lifelines KaplanMeierFitter and a log-rank test via lifelines logrank_test; the stratification is judged correct only if the resulting log-rank test is interpretable (e.g., p < 0.05 for EGFR in TCGA-LGG, with the high-expression group showing worse overall survival, reproducing the paper's Fig. 4D finding). Record group sample sizes alongside the statistic and p-value so the cutoff's balance can be checked.

## Related tools

- **pandas** (Load the TCGA-LGG expression and clinical tables, merge on sample barcode, drop incomplete rows, and compute the mean/median expression cutoff for stratification)
- **lifelines** (Consumes the high/low strata produced by this skill for Kaplan–Meier curve fitting (KaplanMeierFitter) and log-rank testing (logrank_test))

## Examples

```
import pandas as pd; cutoff = merged['EGFR'].median(); merged['group'] = (merged['EGFR'] > cutoff).map({True: 'high', False: 'low'})
```

## Evaluation signals

- Merged cohort contains no rows with missing survival time, vital status, or EGFR expression after the drop step
- Both high- and low-expression strata are non-empty and their sample sizes are recorded in survival_results
- Log-rank test comparing the strata yields p < 0.05 for EGFR in TCGA-LGG, matching the paper's validation criterion
- The high-EGFR group shows worse overall survival on the Kaplan–Meier curves, consistent with the paper's Fig. 4D finding
- Cutoff equals the computed mean/median expression value of the merged cohort (not an externally supplied threshold)

## Limitations

- A mean/median cutoff is a single-cohort binarization; results may differ if quartiles or alternative cutoffs are used
- Dropping samples missing survival time, status, or expression can shift the cohort and hence the cutoff value itself
- The paper's eccDNAs were identified by the AmpliconArchitect pipeline only, so the gene sets available for such stratification analyses are bounded by that pipeline's detections

## Evidence

- [results] Stratify patients into high- and low-EGFR groups using the mean/median EGFR expression value as the cutoff.: [evidence span withheld — anchor status: not_found]
- [methods] Merge the expression and clinical tables on the TCGA sample barcode and drop samples missing survival time, status, or EGFR expression.: [evidence span withheld — anchor status: not_found]
- [results] the task is successful if the log-rank test yields p < 0.05 with the high-EGFR group showing worse overall survival: [evidence span withheld — anchor status: not_found]
- [abstract] Gene expression and clinical data were downloaded from TCGA and CCLE: "Gene expression and clinical data were downloaded from TCGA and CCLE"
- [readme] README describing numbered analysis scripts: "The number notation at the beginning of a script filename indicates the order of an analysis."
