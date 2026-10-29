---
name: one-sided-slope-t-testing
description: Use when you have paired cell-line dependency and copy-number matrices
  (e.g., DepMap DEMETER2 RNAi scores with CCLE CN) and need to identify genes whose
  'essentiality' increases with amplification, e.g., to validate predicted cancer
  drivers.
when_to_use_negative:
- The goal is driver discovery from patient tumor data (iEDGE's cis/trans DE + Sobel
  mediation analysis on TCGA SCNA/expression), not in-silico validation against cell-line
  dependency screens
- Dependency and copy-number matrices cannot be paired by common cell lines, so the
  per-gene regression of dependency on copy number is not well defined
- Testing amplification only is inappropriate — for deletion-driven dependencies the
  one-sided 'alpha < 0' alternative would need to be reversed, which this skill does
  not cover as specified
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: iEDGE
  role: R package providing the run_hyperEnrichment-style one-sided Fisher/hypergeometric
    enrichment test used to test the amplification-driven dependency gene set against
    Rank-1 cis drivers
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Environment in which the per-gene linear regression, one-sided slope t-tests,
    and FDR correction are performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:6c37bf40a56bf66e149234f63a9d9b09f5b0e7ce5f3b4e5924d34f9165d23759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# one-sided-slope-t-testing

## Summary

A one-sided linear-regression slope t-test used in the iEDGE validation workflow to detect amplification-driven gene dependencies: dependency scores from DepMap DEMETER2 RNAi screens are regressed on CCLE copy number, and genes with a significantly negative slope (higher copy number → stronger dependency) are tested for enrichment against iEDGE Rank-1 cis drivers.

## When to use

Use when you have paired cell-line dependency and copy-number matrices (e.g., DepMap DEMETER2 RNAi scores with CCLE CN) and need to identify genes whose 'essentiality' increases with amplification, e.g., to validate predicted cancer drivers.

## When NOT to use

- The goal is driver discovery from patient tumor data (iEDGE's cis/trans DE + Sobel mediation analysis on TCGA SCNA/expression), not in-silico validation against cell-line dependency screens
- Dependency and copy-number matrices cannot be paired by common cell lines, so the per-gene regression of dependency on copy number is not well defined
- Testing amplification only is inappropriate — for deletion-driven dependencies the one-sided 'alpha < 0' alternative would need to be reversed, which this skill does not cover as specified

## Inputs

- DepMap DEMETER2 RNAi dependency score matrix (genes x cell lines)
- CCLE copy-number matrix (genes x cell lines), matched to the same cell lines
- Rank-1 iEDGE cis driver gene list (e.g., from TCGA BRCA)

## Outputs

- Per-gene one-sided slope t-test p-values with FDR correction
- Set of genes with significant negative copy-number-vs-dependency association (amplification-driven dependencies)
- Enrichment test result (one-sided Fisher/hypergeometric p-value) and result table, e.g., depmap_dependency_enrichment.csv

## How to apply

Pair the DepMap DEMETER2 RNAi dependency-score matrix with the CCLE copy-number matrix by cell line. For each gene, fit a linear model of dependency score on copy number and test the slope with a one-sided t-test using the alternative 'alpha < 0' (higher copy number implies stronger dependency). FDR-correct the per-gene t-test p-values across all genes and retain only genes with a significant negative copy-number-vs-dependency association. Feed this significant gene set into the iEDGE one-sided Fisher/hypergeometric enrichment routine (run_hyperEnrichment-style) against the Rank-1 iEDGE cis driver list to validate predicted drivers against DepMap 'essentiality'. In the TCGA breast cancer case, this reproduced a one-sided Fisher test P = 3.53e-5 with the enriched set containing ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1.

## Related tools

- **iEDGE** (R package providing the run_hyperEnrichment-style one-sided Fisher/hypergeometric enrichment test used to test the amplification-driven dependency gene set against Rank-1 cis drivers) — https://github.com/montilab/iEDGE
- **R** (Environment in which the per-gene linear regression, one-sided slope t-tests, and FDR correction are performed)

## Examples

```
fit <- lm(dep ~ cn, data=g); p <- pt(summary(fit)$coefficients[2,3], df=fit$df.residual, lower.tail=TRUE)
```

## Evaluation signals

- The enrichment p-value reproduces the reported one-sided Fisher test P = 3.53e-5 for Rank-1 cis genes in TCGA breast cancer
- The enriched gene set contains ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1
- All tested slope p-values use the one-sided lower-tail alternative (alpha < 0) and are FDR-corrected across genes before thresholding
- The full result table is written to depmap_dependency_enrichment.csv and can be checked for schema completeness (gene, slope p-value, FDR, membership)

## Limitations

- Skill tests amplification-driven dependencies only (one-sided alpha < 0); it does not address deletion-driven dependencies
- Sobel mediation validity and driver prioritization depend on adequate sample size and strong correlation effects, 'conditions which were met in most of the TCGA datasets used in this study' but not all
- A side-by-side comparison with CONEXIC was not possible because its software is not available, limiting external benchmarking of the surrounding iEDGE workflow

## Evidence

- [discussion] DepMap validation with amplification-driven dependencies: [evidence span withheld — anchor status: not_found]
- [methods] Enrichment testing via iEDGE: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [abstract] Reported enrichment finding: "predicted drivers were enriched for functionally relevant cancer genes with amplification-driven dependencies, which are of potential prognostic and therapeutic value"
- [readme] Install iEDGE in R: "install_github("montilab/iEDGE")"
