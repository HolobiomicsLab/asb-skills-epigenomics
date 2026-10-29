---
name: sex-comparative-epigenomic-aging-analysis
description: Use when you have male and female ATAC-seq peak-level log2 fold-change
  tables (young vs old) for the same peak identifiers and need a between-sex correlation
  of aging signatures plus per-sex DA peak counts.
when_to_use_negative:
- Input is single-cell or sorted-cell ATAC-seq data — the paper explicitly notes sex
  differences have not been described at single-cell resolution or in sorted cells,
  so this bulk-PBMC peak-level correlation does not transfer
- You need sex-specific biomarker validation (e.g., IL7R, LEF1, TCF7, IL8, IL18) —
  the paper flags these as unvalidated, requiring future work beyond this correlation
  skill
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: pandas
  role: Load the Fig. 2a Source Data spreadsheet and inner-join male/female peak tables
    on peak identifier
- name: scipy.stats.pearsonr
  role: Compute the between-sex Pearson correlation of young-vs-old log2 fold-change
    vectors with two-sided p-value
- name: R Shiny
  role: Interactive exploration of the aging signatures via the searchable immune-aging
    app
- name: SexDimorphismNatureCommunications
  role: Repository with Scripts (differential analyses), Figures (genomic data figure
    code), and Shiny app code underlying this workflow
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:8a0563c4c7a93288ae0ef54cf641a6c937f6479122dfbde2edb498d285d08f1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - SexDimorphismNatureCommunications
  - pandas
  - scipy.stats.pearsonr
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sex-comparative-epigenomic-aging-analysis

## Summary

A method for quantifying how much of the chromatin accessibility aging signature (ATAC-seq on PBMCs) is shared between men and women by correlating young-vs-old log2 fold-changes per sex at the same peak coordinates, and tallying per-sex opening/closing DA peaks. It reproduces the shared-vs-sex-specific structure of immune system aging reported by Márquez et al. 2020.

## When to use

Use when you have male and female ATAC-seq peak-level log2 fold-change tables (young vs old) for the same peak identifiers and need a between-sex correlation of aging signatures plus per-sex DA peak counts.

## When NOT to use

- Input is single-cell or sorted-cell ATAC-seq data — the paper explicitly notes sex differences have not been described at single-cell resolution or in sorted cells, so this bulk-PBMC peak-level correlation does not transfer
- You need sex-specific biomarker validation (e.g., IL7R, LEF1, TCF7, IL8, IL18) — the paper flags these as unvalidated, requiring future work beyond this correlation skill

## Inputs

- Published Source Data spreadsheet for Fig. 2a (male and female young-vs-old ATAC-seq log2 fold-change per genomic peak coordinate)
- Per-sex peak tables keyed by peak identifier with log2 fold-change values
- Significance thresholds defining 'significant' positive/negative log2 fold change for opening/closing classification

## Outputs

- sex_aging_correlation_results.csv containing the Pearson r, two-sided p-value, and per-sex opening/closing DA peak counts
- Per-sex DA peak tallies (men: 10,196 DA peaks = 5782 closing / 4414 opening; women: 4,516 = 3309 closing / 1207 opening)
- Between-sex Pearson correlation coefficient and p-value of aging log2 fold-change vectors

## How to apply

Load the published Source Data spreadsheet for Fig. 2a with pandas, extracting the young-vs-old log2 fold-change columns for male and female donors keyed by genomic peak coordinates. Inner-join the male and female peak tables on the peak identifier and drop rows with missing log2 fold-change values so both sexes are compared on the identical peak set. Compute the between-sex Pearson correlation of the log2 fold-change vectors with scipy.stats.pearsonr, recording the coefficient and its two-sided p-value. Separately, count differentially accessible peaks per sex, classifying peaks with significant positive log2 fold change as opening and significant negative log2 fold change as closing. Write the correlation, p-value, and per-sex opening/closing totals (men: 5782 closing / 4414 opening of 10,196; women: 3309 closing / 1207 opening of 4,516) to sex_aging_correlation_results.csv. Rationale: the paper's shared epigenomic signature of aging (declining naive T cell, increasing monocyte/cytotoxic loci) is demonstrated exactly by this between-sex correlation of accessibility changes.

## Related tools

- **pandas** (Load the Fig. 2a Source Data spreadsheet and inner-join male/female peak tables on peak identifier)
- **scipy.stats.pearsonr** (Compute the between-sex Pearson correlation of young-vs-old log2 fold-change vectors with two-sided p-value)
- **R Shiny** (Interactive exploration of the aging signatures via the searchable immune-aging app)
- **SexDimorphismNatureCommunications** (Repository with Scripts (differential analyses), Figures (genomic data figure code), and Shiny app code underlying this workflow) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(male_fc.dropna(), female_fc.dropna())
```

## Evaluation signals

- Recomputed Pearson r between male and female log2 fold-change vectors equals 0.445
- Recomputed two-sided p-value for the Pearson correlation is < 0.001
- Male DA peak total is 10,196 (5782 closing / 4414 opening); female total is 4,516 (3309 closing / 1207 opening)
- Inner-join produced a non-empty, sex-paired peak set with no missing log2 fold-change values before correlation
- Output CSV contains correlation coefficient, p-value, and per-sex opening/closing counts with correct column values

## Limitations

- The underlying bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limitation to cell types available in references
- Sex-specific aging signatures described here might be conserved across ethnicities/populations, which requires further investigation
- Male PBMCs underwent greater age-related changes (e.g., 15 times more activated monocyte-specific loci in men), so correlation of shared signatures must be interpreted alongside sex-specific magnitude differences
- Not all sex-biased differences in older adults are attributable to cell composition; monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes needing further study

## Evidence

- [methods] shared correlation workflow: [evidence span withheld — anchor status: not_found]
- [results] reported statistic: [evidence span withheld — anchor status: not_found]
- [abstract] shared signature finding: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [abstract] data scope: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [discussion] annotation caveat: "it is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references."
