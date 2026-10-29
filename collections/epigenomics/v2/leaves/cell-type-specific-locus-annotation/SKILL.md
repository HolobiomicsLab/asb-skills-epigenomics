---
name: cell-type-specific-locus-annotation
description: Use when DA peaks or age-related expression changes from PBMC ATAC-seq/RNA-seq
  must be assigned to cell-type-specific loci to test whether aging signatures (e.g.,
  B cell decline) are shared or sex-specific between men and women.
when_to_use_negative:
- When the analysis targets cell-composition differences measured by flow cytometry
  rather than chromatin accessibility at annotated loci
- When the input lacks cell-type-specific locus annotations (e.g., chromHMM B cell-specific
  labels), since over-representation testing requires them
- When studying aging changes within a single sex only — the key signal here is the
  opposite opening/closing pattern between men and women
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: R Shiny
  role: Interactive exploration of the sex- and cell-type-annotated aging signatures
    via the immune-aging Shiny app
  repo: https://immune-aging.jax.org/
- name: SexDimorphismNatureCommunications
  role: Repository with the analysis code (Scripts, Figures, Shiny) used for the PBMC
    ATAC/RNA/flow analyses behind the locus annotations
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:95c1478efbd74e48629ee9f46005617619349c17910e51d73400072a6efa5fef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - SexDimorphismNatureCommunications
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cell-type-specific-locus-annotation

## Summary

Annotates differentially accessible ATAC-seq peaks and age-related gene expression changes by cell-type specificity (e.g., chromHMM B cell-specific loci) to detect sex-specific immune aging patterns, such as the male-specific decline in B-cell accessibility alongside BCL7A/PAX5/CD79A downregulation.

## When to use

Use when DA peaks or age-related expression changes from PBMC ATAC-seq/RNA-seq must be assigned to cell-type-specific loci to test whether aging signatures (e.g., B cell decline) are shared or sex-specific between men and women.

## When NOT to use

- When the analysis targets cell-composition differences measured by flow cytometry rather than chromatin accessibility at annotated loci
- When the input lacks cell-type-specific locus annotations (e.g., chromHMM B cell-specific labels), since over-representation testing requires them
- When studying aging changes within a single sex only — the key signal here is the opposite opening/closing pattern between men and women

## Inputs

- Source Data Excel workbook for Fig. 2 from the Nature Communications article page (10.1038/s41467-020-14396-9)
- Fig. 2c sheet: DA peak categories (opening/closing) cross-tabulated with chromHMM cell-type-specific loci, stratified by sex
- Fig. 2e sheet: age-related log2 fold-changes with significance annotations for naive B cell genes (BCL7A, PAX5, CD79A) and naive T cell genes (CD27, CCR7, LEF1, TCF7), stratified by sex

## Outputs

- Per-sex Fisher's exact test odds ratios and over-representation statistics for cell-type-specific loci among closing vs opening DA peaks
- sex_specific_aging_verification.csv containing sex, check type, observed direction, and pass/fail for each locus-annotation check
- Verified directional conclusions: male-specific B cell accessibility decline with male-specific BCL7A/PAX5/CD79A downregulation and shared naive T cell gene decline

## How to apply

Download the Source Data workbook for Fig. 2 from the Nature Communications article (10.1038/s41467-020-14396-9) and load the Fig. 2c sheet with pandas. For each sex, test whether chromHMM B cell-specific loci are over-represented among closing versus opening differentially accessible (DA) peaks using Fisher's exact tests in scipy.stats, computing odds ratios as enrichment measures. Confirm directionality: B cell-specific loci should be over-represented among closing peaks in men and among opening peaks in women — the only opposite aging pattern between sexes. Then load the Fig. 2e sheet and extract age-related log2 fold-changes for naive B cell genes (BCL7A, PAX5, CD79A) and naive T cell genes (CD27, CCR7, LEF1, TCF7) stratified by sex, checking the sign and significance annotations of each fold-change. Write all check outcomes (sex, check type, observed direction, pass/fail) to sex_specific_aging_verification.csv.

## Related tools

- **R Shiny** (Interactive exploration of the sex- and cell-type-annotated aging signatures via the immune-aging Shiny app) — https://immune-aging.jax.org/
- **SexDimorphismNatureCommunications** (Repository with the analysis code (Scripts, Figures, Shiny) used for the PBMC ATAC/RNA/flow analyses behind the locus annotations) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
import pandas as pd; from scipy.stats import fisher_exact; df = pd.read_excel('41467_2020_14396_9_MOESM*_SourceData_Fig2.xlsx', sheet_name='Fig2c'); or_m, p_m = fisher_exact(male_table); or_f, p_f = fisher_exact(female_table)
```

## Evaluation signals

- Fisher's exact test shows B cell-specific loci over-represented among closing DA peaks in men (odds ratio > 1 for closing) and among opening DA peaks in women
- Fig. 2e log2 fold-changes for BCL7A, PAX5, and CD79A are negative and significant in men but not in women
- Log2 fold-changes for CD27, CCR7, LEF1, and TCF7 are negative and significant in both sexes (shared naive T cell decline)
- sex_specific_aging_verification.csv is produced with rows covering sex, check type, observed direction, and pass/fail for all loci/gene checks
- Directionality matches the article's finding that B cell loci were 'more likely to be among chromatin-opening peaks in women and among closing peaks in men'

## Limitations

- Bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Sex differences in immune aging have not been described at single-cell resolution or in sorted cells, and functional implications remain unestablished
- Sex-specific aging signatures might be conserved across ethnicities/populations, but this requires further investigation

## Evidence

- [abstract] These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci.: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
- [discussion] Bulk PBMC annotation bias caveat: "it is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references."
- [readme] Repository code organization: "The code can be found under seperate folders: 
1. FlowData: Code used for flow cytometry data analyses
2. Scripts: Scripts for initial data analyses (e.g., differential analyses)"
- [intro] Shiny app sharing: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
