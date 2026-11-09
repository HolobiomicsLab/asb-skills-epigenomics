---
name: pearson-correlation-of-fold-change-signatures
description: Use when two differential analysis results (e.g., young-vs-old log2 fold-change
  per ATAC-seq peak in men vs women) must be compared on a common feature set to measure
  how much of the response is shared versus sex- or condition-specific.
when_to_use_negative:
- The two signatures are not computed over a common, matched feature set (no shared
  peak/gene identifiers to inner-join on), since correlating unmatched feature vectors
  is invalid.
- The goal is single-cell or sorted-cell sex-specific aging analysis, which the paper
  explicitly flags as future work ('Future studies are needed to describe these sex
  differences at single-cell resolution and in sorted cells').
- You need cell-type-resolved rather than bulk PBMC signatures — the paper notes the
  bulk annotation approach 'is prone to biases in references'.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: pandas
  role: Load the Source Data spreadsheet for Fig. 2a and inner-join male/female peak
    tables on peak identifier, dropping rows with missing log2 fold-change values
- name: scipy.stats.pearsonr
  role: Compute the between-sex Pearson correlation coefficient and two-sided p-value
    of the log2 fold-change vectors
- name: R Shiny (immune-aging.jax.org)
  role: Searchable app sharing the paper's aging-signature results that underlie the
    compared signatures
- name: SexDimorphismNatureCommunications repository
  role: Code used for the PBMC ATAC/RNA/flow cytometry analyses, including differential
    analyses scripts and figure code for genomic data analyses
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:6e5a2fc7af5d5a57cc5f7de1487d87ca3d91a8a3392f4e2db74742543cfdc306
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R Shiny (immune-aging.jax.org)
  - SexDimorphismNatureCommunications repository
  - pandas
  - scipy.stats.pearsonr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pearson-correlation-of-fold-change-signatures

## Summary

Quantifies how similar two condition-wise differential accessibility/expression signatures are by Pearson-correlating their log2 fold-change vectors over a shared peak or gene set, as done to compare aging chromatin signatures (young-vs-old ATAC-seq) between male and female PBMC donors (r = 0.445, p < 0.001).

## When to use

Use when two differential analysis results (e.g., young-vs-old log2 fold-change per ATAC-seq peak in men vs women) must be compared on a common feature set to measure how much of the response is shared versus sex- or condition-specific.

## When NOT to use

- The two signatures are not computed over a common, matched feature set (no shared peak/gene identifiers to inner-join on), since correlating unmatched feature vectors is invalid.
- The goal is single-cell or sorted-cell sex-specific aging analysis, which the paper explicitly flags as future work ('Future studies are needed to describe these sex differences at single-cell resolution and in sorted cells').
- You need cell-type-resolved rather than bulk PBMC signatures — the paper notes the bulk annotation approach 'is prone to biases in references'.

## Inputs

- Published Source Data spreadsheet for Fig. 2a (young-vs-old log2 fold-change per ATAC-seq peak for male and female donors)
- Peak identifier table keyed by genomic peak coordinates
- Per-sex differential accessibility significance calls (log2 fold-change sign with significance threshold)

## Outputs

- Pearson correlation coefficient and two-sided p-value between male and female aging log2 fold-change signatures (expected r = 0.445, p < 0.001)
- Per-sex DA peak counts classified as opening vs closing (men: 5782 closing/4414 opening of 10,196; women: 3309 closing/1207 opening of 4,516)
- sex_aging_correlation_results.csv containing the correlation, p-value, and opening/closing counts

## How to apply

Load the published Source Data spreadsheet for Fig. 2a with pandas and extract the young-vs-old log2 fold-change columns for male and female donors, keyed by genomic peak coordinates. Inner-join the male and female peak tables on peak identifier and drop rows with missing log2 fold-change values so both sexes are compared on exactly the same peak set — this join-on-identifier step is essential to avoid inflating or deflating the correlation through unequal feature spaces. Compute the between-sex Pearson correlation of the two log2 fold-change vectors with scipy.stats.pearsonr, recording both the coefficient and its two-sided p-value. In parallel, count differentially accessible peaks per sex, classifying peaks with significant positive log2 fold change as opening and significant negative log2 fold change as closing, to contextualize the shared signature with per-sex DA totals (men: 10,196 DA peaks, 5782 closing/4414 opening; women: 4,516 DA peaks, 3309 closing/1207 opening). Write the correlation coefficient, p-value, and per-sex opening/closing counts to a results CSV (sex_aging_correlation_results.csv) for validation against the reported figures.

## Related tools

- **pandas** (Load the Source Data spreadsheet for Fig. 2a and inner-join male/female peak tables on peak identifier, dropping rows with missing log2 fold-change values)
- **scipy.stats.pearsonr** (Compute the between-sex Pearson correlation coefficient and two-sided p-value of the log2 fold-change vectors)
- **R Shiny (immune-aging.jax.org)** (Searchable app sharing the paper's aging-signature results that underlie the compared signatures)
- **SexDimorphismNatureCommunications repository** (Code used for the PBMC ATAC/RNA/flow cytometry analyses, including differential analyses scripts and figure code for genomic data analyses) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(male_log2fc, female_log2fc)  # after pd.merge(male, female, on='peak_id', how='inner').dropna()
```

## Evaluation signals

- Recomputed Pearson r equals 0.445 with p < 0.001
- Per-sex DA peak totals match the reported figures: 10,196 DA peaks in men (5782 closing / 4414 opening) and 4,516 in women (3309 closing / 1207 opening)
- The correlation is computed only on peaks retained after the inner join on peak identifier with missing log2 fold-change rows dropped — verify the join produced a single shared peak set for both sexes
- Output CSV (sex_aging_correlation_results.csv) contains the correlation coefficient, p-value, and per-sex opening/closing counts in the expected schema

## Limitations

- The shared signature is derived from bulk PBMC ATAC-seq; the paper notes this annotation approach 'is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references'.
- Sex-specific aging signatures may not be conserved across ethnicities/populations, 'which requires further investigation' per the discussion.
- Age-related changes are greater in magnitude in men (e.g., 15 times more monocyte-specific loci activated in men), so fold-change magnitude differences — not just direction — shape the correlation and per-sex DA counts.

## Evidence

- [results] Male and female epigenomic aging signatures are significantly correlated (Pearson r = 0.445, p < 0.001): [evidence span withheld — anchor status: not_found]
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry"
- [readme] README structure of analysis code: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
- [discussion] bulk PBMC annotation bias caveat: "it is prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references."
