---
name: sex-stratified-aging-signature-interpretation
description: Use when comparing age-related chromatin accessibility or gene-expression
  changes between male and female donors to determine whether cell-type-specific loci
  (e.g., B cell loci) close in one sex and open in the other.
when_to_use_negative:
- When the goal is shared (non-sex-stratified) aging signatures, such as the common
  decline in naive T cell loci — that is a single-cohort trend, not a male-vs-female
  contrast
- When input is single-cell or sorted-cell data — the article notes its conclusions
  come from bulk PBMC annotation and that sex differences 'at single-cell resolution
  or in sorted cells' remain unestablished
- When comparing cell-composition shifts rather than chromatin accessibility at annotated
  cell-type-specific loci — use flow cytometry deconvolution instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: R Shiny
  role: Searchable application (https://immune-aging.jax.org/) for exploring the sex-specific
    aging signatures that this verification skill checks
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:24e481c3b93c7ea3466474fbd5bcdfabfbb70ccc98221eb237ce8daab5f3fd9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sex-stratified-aging-signature-interpretation

## Summary

Method for verifying sex-specific (male vs female) aging signatures in PBMC ATAC-seq/RNA-seq data by testing whether chromHMM cell-type-specific loci (e.g., B cell loci BCL7A, PAX5, CD79A) are enriched among closing DA peaks in men versus opening DA peaks in women, using Fisher's exact tests on the article's Source Data sheets.

## When to use

Use when comparing age-related chromatin accessibility or gene-expression changes between male and female donors to determine whether cell-type-specific loci (e.g., B cell loci) close in one sex and open in the other.

## When NOT to use

- When the goal is shared (non-sex-stratified) aging signatures, such as the common decline in naive T cell loci — that is a single-cohort trend, not a male-vs-female contrast
- When input is single-cell or sorted-cell data — the article notes its conclusions come from bulk PBMC annotation and that sex differences 'at single-cell resolution or in sorted cells' remain unestablished
- When comparing cell-composition shifts rather than chromatin accessibility at annotated cell-type-specific loci — use flow cytometry deconvolution instead

## Inputs

- Source Data workbook (Fig. 2 sheets) from the Nature Communications article page (10.1038/s41467-020-14396-9)
- Fig. 2c sheet: chromHMM cell-type-specific loci mapped to male/female opening vs closing DA peaks
- Fig. 2e sheet: age-related log2 fold-changes with sex stratification and significance annotations for B cell (BCL7A, PAX5, CD79A) and T cell (CD27, CCR7, LEF1, TCF7) genes

## Outputs

- Per-sex Fisher's exact test odds ratios and p-values for enrichment of B cell-specific loci among closing vs opening DA peaks
- Table of log2 fold-changes and significance annotations for the seven naive marker genes, stratified by sex
- sex_specific_aging_verification.csv with columns: sex, check type, observed direction, pass/fail

## How to apply

Download the Source Data workbook for Fig. 2 from the Nature Communications article (DOI 10.1038/s41467-020-14396-9) and load the Fig. 2c sheet with pandas. For each sex, build a 2x2 contingency table of chromHMM B cell-specific loci versus other loci among closing versus opening differentially accessible (DA) peaks, and compute the over-representation (odds ratio) with scipy.stats Fisher's exact test. Confirm directionality: B cell-specific loci over-represented among closing peaks in men and among opening peaks in women — the only opposite aging pattern between sexes. Then load the Fig. 2e sheet and extract age-related log2 fold-changes for naive B cell genes (BCL7A, PAX5, CD79A) and naive T cell genes (CD27, CCR7, LEF1, TCF7) stratified by sex, checking sign and significance annotations to verify male-specific B cell gene downregulation versus T cell decline in both sexes. Record all check outcomes (sex, check type, observed direction, pass/fail) to sex_specific_aging_verification.csv.

## Related tools

- **R Shiny** (Searchable application (https://immune-aging.jax.org/) for exploring the sex-specific aging signatures that this verification skill checks) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
sd = pd.read_excel('Fig2_SourceData.xlsx', sheet_name='Fig2c'); or_, p = scipy.stats.fisher_exact([[bcell_closing, other_closing], [bcell_opening, other_opening]])
```

## Evaluation signals

- Fisher's exact test on Fig. 2c must show B cell-specific loci over-represented among CLOSING peaks in men and among OPENING peaks in women — reversal of direction between sexes is the pass condition
- B cell genes BCL7A, PAX5, CD79A must show negative age-related log2 fold-changes with significance in men (male-specific downregulation), while CD27, CCR7, LEF1, TCF7 decline in both sexes
- Every row in sex_specific_aging_verification.csv must record sex, check type, observed direction, and pass/fail; no missing direction annotations
- Effect-magnitude asymmetry is expected: changes are 'greater in magnitude in men' (e.g., 15x more activated monocyte-specific loci in men), so female-side enrichment should be weaker than male-side closing
- Cross-check fold-change signs against the R Shiny app (immune-aging.jax.org) and code in the UcarLab/SexDimorphismNatureCommunications repository Figures scripts

## Limitations

- Bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Findings are from 172 healthy adults aged 22-93; conservation of sex-specific aging signatures across ethnicities/populations requires further investigation
- Functional implications and cell-intrinsic mechanisms of the sex differences (e.g., sex hormones) remain unestablished

## Evidence

- [abstract] male-specific decline in B-cell specific loci: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
- [abstract] cohort and assays: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] Shiny app for exploration: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [discussion] bulk annotation bias: "it is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references."
- [readme] repository scope: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
