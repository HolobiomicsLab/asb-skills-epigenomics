---
name: one-sided-fisher-gene-set-enrichment
description: Use when validating a ranked or predicted driver gene list by testing
  its overlap against a curated functional gene set, e.g., reproducing the P = 3.53e-5
  enrichment of BRCA Rank-1 cis genes among amplification-driven dependency genes.
when_to_use_negative:
- The gene list has not been ranked or filtered by a driver-prediction step (e.g.,
  iEDGE mediation analysis), so enrichment lacks a meaningful hypothesis
- The gene universe (ntotal) or set membership is undefined, since Fisher exact counts
  depend critically on the correct background
- The functional gene set is already the same compendium used to build the driver
  list, making enrichment circular
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: Provides run_hyperEnrichment for one-sided Fisher/hypergeometric enrichment
    of predicted cis drivers against functional gene categories
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Implementation environment for the Fisher test and for the upstream linear
    regression of DepMap dependency scores on CCLE copy number
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:55aee7bebe1e3e09ad54dabfb3912540d38ec765d6945a692d9027b0acecaa4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# One-sided Fisher gene-set enrichment

## Summary

Tests whether a candidate driver gene list (e.g., iEDGE Rank-1 cis genes from TCGA breast cancer) is significantly over-represented in a functional gene set (e.g., amplification-driven DepMap dependency genes) using a one-sided Fisher exact / hypergeometric test as implemented in iEDGE's run_hyperEnrichment.

## When to use

Use when validating a ranked or predicted driver gene list by testing its overlap against a curated functional gene set, e.g., reproducing the P = 3.53e-5 enrichment of BRCA Rank-1 cis genes among amplification-driven dependency genes.

## When NOT to use

- The gene list has not been ranked or filtered by a driver-prediction step (e.g., iEDGE mediation analysis), so enrichment lacks a meaningful hypothesis
- The gene universe (ntotal) or set membership is undefined, since Fisher exact counts depend critically on the correct background
- The functional gene set is already the same compendium used to build the driver list, making enrichment circular

## Inputs

- List of predicted driver genes (drawn), e.g., iEDGE Rank-1 cis genes from TCGA BRCA
- Functional gene set (categories), e.g., genes with significant negative copy-number-vs-dependency association from DepMap DEMETER2 RNAi + CCLE copy number regression
- Total number of genes tested (ntotal) defining the gene universe
- GMT gene set files (e.g., h.all.v5.0.symbols.gmt) when enriching against pathway databases

## Outputs

- Enrichment table with per-set overlap counts, hypergeometric/Fisher p-values and corrected p-values (depmap_dependency_enrichment.csv)
- Identity of enriched functional gene sets containing expected drivers (ERBB2, CCNE1, CCND1, FOXA1, ANKRD17, MCL1)

## How to apply

First derive the background gene universe (ntotal) and the functional gene set (categories) — here, genes whose DepMap DEMETER2 RNAi dependency scores are significantly negatively regressed on CCLE copy number across paired cell lines (one-sided t-test, slope < 0, FDR-corrected). Then run the iEDGE hypergeometric enrichment with the predicted driver list as 'drawn', respecting min.drawsize and multiple-testing (mht) settings per the run_hyperEnrichment signature. The expected positive control is that the enriched set contains ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1, with one-sided Fisher P = 3.53e-5 for BRCA Rank-1 cis genes, matching the article's in silico validation of predicted drivers against DepMap essentiality data. Write full results (overlap size, set size, odds ratio, p-value) to depmap_dependency_enrichment.csv.

## Related tools

- **iEDGE** (Provides run_hyperEnrichment for one-sided Fisher/hypergeometric enrichment of predicted cis drivers against functional gene categories) — https://github.com/montilab/iEDGE
- **R** (Implementation environment for the Fisher test and for the upstream linear regression of DepMap dependency scores on CCLE copy number)

## Examples

```
library(iEDGE); res <- run_hyperEnrichment(drawn = rank1_cis_genes, categories = amp_dependency_genes, ntotal = ntotal, min.drawsize = 5, mht = "BH", verbose = TRUE)
```

## Evaluation signals

- Reproduced enrichment p-value for BRCA Rank-1 cis genes vs amplification-driven dependency genes equals the reported 3.53e-5 (one-sided Fisher test)
- Enriched gene set contains the expected drivers ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1
- run_hyperEnrichment is called with a well-defined ntotal gene universe and non-empty 'drawn' list; no sets below min.drawsize are reported
- Output CSV depmap_dependency_enrichment.csv contains p-value columns and passes multiple-testing (mht) correction without NA rows

## Limitations

- Sobel test validity — and by extension the cis-driver rankings being enriched — depends on adequate sample size and strong correlation effects, conditions not met in all TCGA datasets
- Assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases, weakening the enriched driver list
- Side-by-side validation against alternative methods such as CONEXIC was not possible because its software is not available

## Evidence

- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [discussion] DepMap validation rationale: [evidence span withheld — anchor status: not_found]
- [abstract] amplification-driven dependency finding: "predicted drivers were enriched for functionally relevant cancer genes with amplification-driven dependencies, which are of potential prognostic and therapeutic value"
- [readme] iEDGE package description: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA
and gene expression data"
- [discussion] driver databases enrichment validation: "first testing for their enrichment with known drivers from multiple cancer driver databases"
