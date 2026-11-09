---
name: source-data-reproduction-check
description: Use when a target finding is tied to a specific figure of a paper with
  a downloadable Source Data file, and you must verify a directional claim (e.g. male-specific
  B cell decline) from the published numbers.
when_to_use_negative:
- The paper provides no Source Data workbook or figure-level sheets — raw data re-analysis
  (e.g. re-running differential accessibility from FASTQs) is required instead.
- The claim depends on cell-type-composition or flow cytometry results not contained
  in the figure sheets (e.g. monocyte activation differences discussed in the paper's
  discussion).
- You need single-cell or sorted-cell resolution; the article's discussion explicitly
  notes sex differences have not yet been described at single-cell resolution or in
  sorted cells.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: R Shiny
  role: Searchable web application (https://immune-aging.jax.org/) where the article's
    age-related genomic results are shared and can be cross-checked against the Source
    Data values
- name: SexDimorphismNatureCommunications
  role: GitHub repository with the analysis code (Scripts, Figures, Shiny folders)
    used for the PBMC ATAC/RNA/flow analyses behind the figures being reproduced
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:66ca2be1e1b8212942ef0f7e81fcc0db84f5ff4ef347f3d32a88e3b0e5f3a4e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UcarLab/SexDimorphismNatureCommunications
  tools:
  - R Shiny
  - SexDimorphismNatureCommunications
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# source-data-reproduction-check

## Summary

Reproduces a published figure's key statistical claims by downloading the article's Source Data workbook (e.g. Fig. 2 from the Nature Communications paper 10.1038/s41467-020-14396-9), re-running enrichment and fold-change checks (Fisher's exact test, log2FC sign/significance) in pandas/scipy, and writing a structured pass/fail verification report.

## When to use

Use when a target finding is tied to a specific figure of a paper with a downloadable Source Data file, and you must verify a directional claim (e.g. male-specific B cell decline) from the published numbers.

## When NOT to use

- The paper provides no Source Data workbook or figure-level sheets — raw data re-analysis (e.g. re-running differential accessibility from FASTQs) is required instead.
- The claim depends on cell-type-composition or flow cytometry results not contained in the figure sheets (e.g. monocyte activation differences discussed in the paper's discussion).
- You need single-cell or sorted-cell resolution; the article's discussion explicitly notes sex differences have not yet been described at single-cell resolution or in sorted cells.

## Inputs

- Source Data workbook (Excel) from the Nature Communications article page for 10.1038/s41467-020-14396-9, sheets Fig. 2c and Fig. 2e
- Expected direction/significance annotations from the paper's findings (e.g. BCL7A/PAX5/CD79A downregulated only in men)

## Outputs

- Per-sex enrichment statistics (odds ratios from Fisher's exact test) for chromHMM B cell-specific loci among closing vs opening DA peaks
- Extracted log2 fold-changes and significance annotations for BCL7A, PAX5, CD79A, CD27, CCR7, LEF1, TCF7 stratified by sex
- sex_specific_aging_verification.csv with columns sex, check type, observed direction, pass/fail

## How to apply

Download the Source Data workbook from the article page (DOI 10.1038/s41467-020-14396-9) and load the relevant sheet — Fig. 2c for the chromatin-accessibility enrichment and Fig. 2e for gene-level log2 fold-changes — with pandas. For Fig. 2c, build 2x2 contingency tables per sex crossing chromHMM B cell-specific loci versus other loci against closing versus opening differentially accessible (DA) peaks, and compute over-representation with scipy.stats.fisher_exact; confirm B cell loci are over-represented among closing peaks in men and among opening peaks in women. For Fig. 2e, extract age-related log2 fold-changes stratified by sex for the naive B cell genes BCL7A, PAX5, CD79A and the naive T cell genes CD27, CCR7, LEF1, TCF7, then check sign and significance annotations to verify male-specific B cell downregulation versus decline in both sexes for T cell genes. Record each check (sex, check type, observed direction, pass/fail) to a CSV such as sex_specific_aging_verification.csv.

## Related tools

- **R Shiny** (Searchable web application (https://immune-aging.jax.org/) where the article's age-related genomic results are shared and can be cross-checked against the Source Data values)
- **SexDimorphismNatureCommunications** (GitHub repository with the analysis code (Scripts, Figures, Shiny folders) used for the PBMC ATAC/RNA/flow analyses behind the figures being reproduced) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
import pandas as pd; from scipy.stats import fisher_exact; df = pd.read_excel('SourceData_Fig2.xlsx', sheet_name='Fig. 2c'); odds, p = fisher_exact([[b_closing, other_closing],[b_opening, other_opening]])
```

## Evaluation signals

- Fisher's exact test on the Fig. 2c sheet shows B cell-specific loci over-represented among closing DA peaks in men and among opening DA peaks in women (opposite directions between sexes).
- Fig. 2e log2 fold-changes for BCL7A, PAX5, and CD79A are negative (downregulated) with significance annotation in men but not in women.
- Fig. 2e log2 fold-changes for CD27, CCR7, LEF1, and TCF7 are negative in both sexes, consistent with the shared declining naive T cell signature.
- sex_specific_aging_verification.csv is written with exactly the fields sex, check type, observed direction, and pass/fail, and all expected checks pass.
- Recovered enrichment directions match the article's stated finding that changes are 'greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci'.

## Limitations

- The check relies on the aggregated numbers published in the Source Data workbook; it does not re-derive differential accessibility from the underlying ATAC-seq/RNA-seq data of the 172 healthy adults aged 22-93.
- The bulk PBMC annotation approach used by the paper is prone to biases in reference datasets (differences in data quality and limited available cell types), which propagates to figure-level values.
- Verification of directionality/significance depends on the annotations present in the workbook sheets; if annotations are absent, pass/fail judgements are incomplete.

## Evidence

- [abstract] These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci.: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry.: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/).: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
