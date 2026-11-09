---
name: chromatin-accessibility-enrichment-analysis
description: Use when ATAC-seq DA peak sets (opening vs. closing with age) must be
  tested for enrichment of cell-type-specific chromatin annotations per sex, e.g.
  to verify male-specific B cell locus decline or female opening patterns.
when_to_use_negative:
- The question concerns shared (non-sex-stratified) aging signatures such as declining
  naive T cell or increasing monocyte/cytotoxic functions, where a single pooled enrichment
  test suffices
- Input is bulk gene expression without ATAC-seq DA peak direction labels — the Fisher
  test requires opening/closing peak contingency counts
- Analysis needs single-cell or sorted-cell resolution, which the authors explicitly
  flag as unestablished for these sex differences
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R Shiny
  role: Searchable web application (https://immune-aging.jax.org/) used to share and
    explore the epigenomic aging results underlying the enrichment checks
  repo: https://immune-aging.jax.org/
- name: pandas / scipy.stats
  role: Python libraries used to load Fig. 2c/2e sheets of the Source Data workbook
    and run Fisher's exact tests for locus enrichment
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:a97c86830baa686d85140198b414f74ea98ad180b50bacc6590f3ada3f0bc1aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas / scipy.stats
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chromatin-accessibility-enrichment-analysis

## Summary

Tests whether functionally annotated loci (e.g., chromHMM B cell-specific loci) are over-represented among age-related closing vs. opening differentially accessible ATAC-seq peaks, stratified by sex, to detect sex-specific epigenomic decline such as the male-specific B cell accessibility loss reported for PBMCs from 172 healthy adults aged 22–93.

## When to use

Use when ATAC-seq DA peak sets (opening vs. closing with age) must be tested for enrichment of cell-type-specific chromatin annotations per sex, e.g. to verify male-specific B cell locus decline or female opening patterns.

## When NOT to use

- The question concerns shared (non-sex-stratified) aging signatures such as declining naive T cell or increasing monocyte/cytotoxic functions, where a single pooled enrichment test suffices
- Input is bulk gene expression without ATAC-seq DA peak direction labels — the Fisher test requires opening/closing peak contingency counts
- Analysis needs single-cell or sorted-cell resolution, which the authors explicitly flag as unestablished for these sex differences

## Inputs

- Source Data workbook (Excel) for Fig. 2 of DOI 10.1038/s41467-020-14396-9, Fig. 2c sheet with chromHMM locus-class x DA-peak-direction counts per sex
- Fig. 2e sheet with age-related log2 fold-changes and significance annotations for BCL7A, PAX5, CD79A, CD27, CCR7, LEF1, TCF7 by sex
- ATAC-seq DA peak sets (opening vs. closing with age) and chromHMM cell-type-specific locus annotations from PBMCs of 172 healthy adults aged 22-93

## Outputs

- Per-sex Fisher's exact test odds ratios and p-values for enrichment of B cell-specific loci among closing vs. opening DA peaks
- sex_specific_aging_verification.csv with columns: sex, check type, observed direction, pass/fail

## How to apply

Download the Source Data workbook for Fig. 2 of the Nature Communications article (10.1038/s41467-020-14396-9) and load the Fig. 2c sheet with pandas. For each sex separately, build a 2x2 contingency table of chromHMM B cell-specific loci vs. other loci crossed with closing vs. opening differentially accessible (DA) peaks and compute the odds ratio / over-representation with scipy.stats.fisher_exact. Confirm directionality: B cell-specific loci over-represented among closing peaks in men and among opening peaks in women. For RNA-seq cross-validation, load the Fig. 2e sheet and extract age-related log2 fold-changes for naive B cell genes BCL7A, PAX5, CD79A and naive T cell genes CD27, CCR7, LEF1, TCF7, stratified by sex, checking the sign and significance annotations (male-specific downregulation of B cell genes; T cell gene decline in both sexes). Write all check outcomes (sex, check type, observed direction, pass/fail) to sex_specific_aging_verification.csv.

## Related tools

- **R Shiny** (Searchable web application (https://immune-aging.jax.org/) used to share and explore the epigenomic aging results underlying the enrichment checks) — https://immune-aging.jax.org/
- **pandas / scipy.stats** (Python libraries used to load Fig. 2c/2e sheets of the Source Data workbook and run Fisher's exact tests for locus enrichment) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
import pandas as pd; from scipy.stats import fisher_exact; df = pd.read_excel('Source_Data_Fig2.xlsx', sheet_name='Fig. 2c'); OR, p = fisher_exact([[bcell_closing, other_closing],[bcell_opening, other_opening]])
```

## Evaluation signals

- Fisher's exact test on Fig. 2c counts yields over-representation of B cell-specific loci among closing DA peaks in men and among opening DA peaks in women (opposite directions between sexes)
- Fig. 2e log2 fold-changes show negative (downregulated) significant values for BCL7A, PAX5, and CD79A in men but not women
- Fig. 2e log2 fold-changes for CD27, CCR7, LEF1, TCF7 show decline in both sexes, matching the shared declining naive T cell signature
- sex_specific_aging_verification.csv is populated with rows for both sexes covering check type, observed direction, and pass/fail without missing cells
- Enrichment magnitudes align with the report that male PBMCs underwent greater age-related changes, e.g. 15 times more activated monocyte-specific loci in men

## Limitations

- Sex-specific aging signatures described might be conserved across ethnicities/populations, which requires further investigation
- The bulk PBMC annotation approach is prone to biases in references, e.g. differences in data quality and limitation to cell types available in references
- Functional implications of the observed sex-specific accessibility changes are not established; future studies at single-cell or sorted-cell resolution are needed
- The repository README notes code is organized in separate folders (FlowData, Scripts, Figures, Shiny) and links a preprint (biorxiv 755702) rather than the final journal version

## Evidence

- [abstract] These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci.: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
- [intro] Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends.: "Computational pipelines for functional enrichments, trajectory and breakpoint analyses revealed immune system aging signatures including longitudinal genomic trends."
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry.: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
