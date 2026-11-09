---
name: copy-number-vs-dependency-regression
description: Use when in silico validation of predicted cancer drivers requires linking
  somatic copy-number amplifications to gene essentiality from DepMap RNAi screens
  and CCLE copy-number data.
when_to_use_negative:
- When the dependency and copy-number data cannot be paired by shared cell line identifiers
  (misaligned CCLE/DepMap sample sets)
- When the driver list to be validated is not ranked from cis mediation analysis,
  so a Rank-1 'drawn' set for run_hyperEnrichment cannot be defined
- When the question concerns trans-driver pathway enrichment (GMT-based h.all/c2.cp/c3.tft
  testing) rather than dependency-based validation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: iEDGE
  role: R package providing run_hyperEnrichment for the one-sided Fisher/hypergeometric
    enrichment of the amplified-dependent gene set against Rank-1 cis drivers
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Environment for loading the DEMETER2 and CCLE matrices, fitting per-gene lm
    regressions, and running one-sided t-tests with FDR correction
- name: DepMap
  role: Source of RNAi genetic screen (DEMETER2) essentiality data used as the validation
    compendium
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:5deeb3315a7b231bcbc71a3bbcc1c6faf53d4008b8ecd59cca45c6baa29eaa35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - DepMap
  - R
  - iEDGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# copy-number-vs-dependency-regression

## Summary

A statistical validation workflow that regresses DepMap DEMETER2 RNAi dependency scores on CCLE copy number per gene (one-sided t-test on the slope, alpha < 0) to identify amplification-driven dependencies, then tests their enrichment against iEDGE Rank-1 cis drivers via run_hyperEnrichment-style Fisher/hypergeometric testing.

## When to use

Use when in silico validation of predicted cancer drivers requires linking somatic copy-number amplifications to gene essentiality from DepMap RNAi screens and CCLE copy-number data.

## When NOT to use

- When the dependency and copy-number data cannot be paired by shared cell line identifiers (misaligned CCLE/DepMap sample sets)
- When the driver list to be validated is not ranked from cis mediation analysis, so a Rank-1 'drawn' set for run_hyperEnrichment cannot be defined
- When the question concerns trans-driver pathway enrichment (GMT-based h.all/c2.cp/c3.tft testing) rather than dependency-based validation

## Inputs

- DepMap DEMETER2 RNAi dependency score matrix (genes x cell lines)
- CCLE copy-number matrix (genes x cell lines), paired with the dependency matrix by cell line identifier
- iEDGE Rank-1 cis driver gene list (e.g. from TCGA breast cancer analysis)
- Background universe gene count (ntotal) for the hypergeometric test

## Outputs

- Per-gene regression slopes, one-sided t-test p-values, and FDR-corrected q-values for copy-number-vs-dependency association
- Set of genes with significant negative copy-number-vs-dependency association (amplification-driven dependencies)
- Enrichment test p-value of amplification-driven dependency genes against the Rank-1 cis driver list (reported: P = 3.53e-5)
- Full results table, e.g. depmap_dependency_enrichment.csv

## How to apply

Load the DepMap DEMETER2 RNAi dependency score matrix and the CCLE copy-number matrix and pair them by cell line. For each gene, fit a linear regression of dependency score on copy number and test the slope with a one-sided t-test (alternative: alpha < 0, i.e. higher copy number implies stronger dependency). FDR-correct the t-test p-values across genes and retain the significant negative copy-number-vs-dependency gene set. Then perform a one-sided Fisher exact / hypergeometric enrichment test (iEDGE run_hyperEnrichment-style) of that significant set against the iEDGE Rank-1 cis driver list, using the run_hyperEnrichment parameters (drawn, categories, ntotal, min.drawsize, mht). The rationale follows the iEDGE paper's characterization of predicted drivers by their essentiality against DepMap genetic screen data.

## Related tools

- **iEDGE** (R package providing run_hyperEnrichment for the one-sided Fisher/hypergeometric enrichment of the amplified-dependent gene set against Rank-1 cis drivers) — https://github.com/montilab/iEDGE
- **R** (Environment for loading the DEMETER2 and CCLE matrices, fitting per-gene lm regressions, and running one-sided t-tests with FDR correction)
- **DepMap** (Source of RNAi genetic screen (DEMETER2) essentiality data used as the validation compendium)

## Examples

```
s <- summary(lm(dep ~ cn, data = g)); p_one_sided <- pt(s$coefficients[2, 3], df = s$df, lower.tail = TRUE)  # slope t-test, alpha < 0 per gene
```

## Evaluation signals

- The enrichment one-sided Fisher/hypergeometric p-value reproduces the reported P = 3.53e-5 for TCGA breast cancer Rank-1 cis genes
- The enriched amplification-driven dependency gene set contains ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1
- Per-gene slope t-tests are one-sided with alternative alpha < 0, and retained genes have significant FDR-corrected p-values
- Results are written in full to depmap_dependency_enrichment.csv with per-gene regression and enrichment rows
- The DEMETER2 and CCLE matrices are verified to be paired on the same cell lines before regression

## Limitations

- Further experimental studies are needed to validate and characterize predicted drivers; this is an in silico validation against DepMap screens only
- The directionality assumption (higher copy number implies stronger dependency, alpha < 0) must hold; deletions and neutral alterations are not addressed
- Sobel/mediation-based driver rankings used as the 'drawn' set are only valid when sample size and correlation effects are adequate, conditions not met in all TCGA datasets

## Evidence

- [discussion] DepMap discussion: [evidence span withheld — anchor status: not_found]
- [abstract] amplification-driven dependency enrichment finding: "predicted drivers were enriched for functionally relevant cancer genes with amplification-driven dependencies, which are of potential prognostic and therapeutic value"
- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [readme] iEDGE package availability: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
