---
name: atac-seq-differential-accessibility-peak-classification
description: Use when comparing two ATAC-seq aging signatures (e.g., male vs female
  donors) on a shared peak set, needing a between-sex correlation of log2 fold-changes
  plus per-sex opening/closing DA peak counts.
when_to_use_negative:
- The analysis requires cell-type-resolved or single-cell differential accessibility
  — the article notes future studies are needed 'at single-cell resolution and in
  sorted cells', so bulk-PBMC peak aggregation is the wrong granularity.
- You need sex-specific biomarker validation or mechanistic explanation of differences
  (e.g., sex hormones, DAMPs/PAMPs) — the article explicitly leaves these unestablished.
- Input data is already a correlation matrix or aggregated summary rather than per-peak
  log2 fold-change values keyed by peak coordinates.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0625
tools:
- name: pandas
  role: Load the Fig. 2a Source Data spreadsheet and inner-join male and female peak
    tables on peak identifier
- name: scipy.stats.pearsonr
  role: Compute the between-sex Pearson correlation coefficient and two-sided p-value
    of young-vs-old log2 fold-change vectors
- name: R Shiny
  role: Interactive web application (https://immune-aging.jax.org/) sharing the sex-dimorphism
    aging results for exploration
- name: UcarLab/SexDimorphismNatureCommunications
  role: Repository containing the analysis code, including Scripts for differential
    analyses and Figures code for genomic analyses
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:b22a322d4dcd7b505fa997c04dd0fdbf55568e781686b6c153c5b0dc2e8c4b99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - UcarLab/SexDimorphismNatureCommunications
  - pandas
  - scipy.stats.pearsonr
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# atac-seq-differential-accessibility-peak-classification

## Summary

Recompute and validate sex-shared vs sex-specific ATAC-seq aging signatures in PBMCs by joining male and female young-vs-old log2 fold-change peak tables on peak coordinates, computing the between-sex Pearson correlation (expected r = 0.445, p < 0.001), and classifying differentially accessible (DA) peaks per sex as opening or closing.

## When to use

Use when comparing two ATAC-seq aging signatures (e.g., male vs female donors) on a shared peak set, needing a between-sex correlation of log2 fold-changes plus per-sex opening/closing DA peak counts.

## When NOT to use

- The analysis requires cell-type-resolved or single-cell differential accessibility — the article notes future studies are needed 'at single-cell resolution and in sorted cells', so bulk-PBMC peak aggregation is the wrong granularity.
- You need sex-specific biomarker validation or mechanistic explanation of differences (e.g., sex hormones, DAMPs/PAMPs) — the article explicitly leaves these unestablished.
- Input data is already a correlation matrix or aggregated summary rather than per-peak log2 fold-change values keyed by peak coordinates.

## Inputs

- Source Data spreadsheet for Fig. 2a (published supplementary table of young-vs-old log2 fold-change per peak, split by sex)
- Male PBMC ATAC-seq peak table with log2 fold-change per genomic peak coordinate
- Female PBMC ATAC-seq peak table with log2 fold-change per genomic peak coordinate

## Outputs

- sex_aging_correlation_results.csv containing the Pearson r, two-sided p-value, and per-sex DA peak counts (opening/closing)
- Between-sex Pearson correlation of log2 fold-changes (r = 0.445, p < 0.001)
- Per-sex DA peak tallies: 10,196 DA peaks in men (5782 closing / 4414 opening); 4,516 in women (3309 closing / 1207 opening)

## How to apply

Load the published Source Data spreadsheet for Fig. 2a with pandas, extracting the young-vs-old log2 fold-change columns for male and female donors keyed by genomic peak coordinates. Inner-join the male and female peak tables on the peak identifier so both sexes are compared on the same peak set, dropping rows with missing log2 fold-change values. Compute the between-sex Pearson correlation with scipy.stats.pearsonr on the two log2 fold-change vectors, recording the coefficient and two-sided p-value. Count DA peaks per sex by classifying peaks with significant positive log2 fold change as opening and significant negative log2 fold change as closing, then assemble totals (men: 5782 closing / 4414 opening of 10,196; women: 3309 closing / 1207 opening of 4,516). Write the correlation coefficient, p-value, and per-sex opening/closing counts to a results CSV. The rationale is that a significantly positive between-sex correlation demonstrates a shared epigenomic aging signature — declining naive T cell and increasing monocyte/cytotoxic loci — while the asymmetric DA totals (more DA peaks in men) quantify the greater magnitude of change in male PBMCs.

## Related tools

- **pandas** (Load the Fig. 2a Source Data spreadsheet and inner-join male and female peak tables on peak identifier)
- **scipy.stats.pearsonr** (Compute the between-sex Pearson correlation coefficient and two-sided p-value of young-vs-old log2 fold-change vectors)
- **R Shiny** (Interactive web application (https://immune-aging.jax.org/) sharing the sex-dimorphism aging results for exploration)
- **UcarLab/SexDimorphismNatureCommunications** (Repository containing the analysis code, including Scripts for differential analyses and Figures code for genomic analyses) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(male_fc, female_fc)  # male_fc, female_fc: inner-joined young-vs-old log2 fold-change vectors from Fig. 2a Source Data; expect r=0.445, p<0.001
```

## Evaluation signals

- Recomputed Pearson r between male and female log2 fold-change vectors equals 0.445 with p < 0.001.
- Per-sex DA peak totals match reported figures exactly: 10,196 DA peaks in men (5782 closing / 4414 opening) and 4,516 in women (3309 closing / 1207 opening).
- The inner-join on peak identifier yields the same peak set for both sexes with no missing log2 fold-change rows remaining.
- Output CSV (sex_aging_correlation_results.csv) contains correlation coefficient, p-value, and opening/closing counts per sex with a valid schema.
- Closing peaks outnumber opening peaks in both sexes, consistent with the reported directional imbalance.

## Limitations

- Changes are of greater magnitude in men (more DA peaks, 15 times more activated monocyte-specific loci in men), so per-sex counts are not symmetric and the correlation r = 0.445 reflects sharing with substantial sex-specific components.
- Bulk PBMC annotation is 'prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references', so DA peaks may reflect cell-composition shifts rather than cell-intrinsic chromatin changes.
- Not all sex-biased differences in older adults are attributable to cell composition; monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes the peak-level fold-changes alone cannot disentangle.

## Evidence

- [abstract] Shared epigenomic signature of aging with declining naive T cell and increasing monocyte and cytotoxic cell functions: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [abstract] Greater magnitude of change in men: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
- [abstract] Cohort and assays underlying the peak tables: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [discussion] Bulk annotation bias caveat: "it is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references."
- [readme] Repository structure for analysis code: "2. Scripts: Scripts for initial data analyses (e.g., differential analyses)"
- [intro] R Shiny sharing app: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
