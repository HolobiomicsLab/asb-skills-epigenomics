---
name: rna-essentiality-screen-analysis
description: Use when a ranked list of candidate cis driver genes (e.g., TCGA Rank-1
  iEDGE genes) must be validated against DepMap RNAi essentiality screens where dependency
  scores correlate with copy number amplification.
when_to_use_negative:
- When the candidate gene list is already validated against known driver compendia
  and no functional essentiality evidence is required
- When RNAi dependency data are not matched to copy number data for the same cell
  lines — the slope test requires paired DepMap/CCLE samples
- When the candidate set derives from trans effects rather than cis (amplification-linked)
  driver predictions, since the rationale is amplification-driven dependency
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: Provides run_hyperEnrichment, the one-sided hypergeometric/Fisher enrichment
    test used to test the candidate cis driver list against amplification-driven dependency
    gene sets
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Implementation language for the linear regression slope tests, FDR correction,
    and enrichment workflow
- name: DepMap
  role: Source of RNAi genetic screens and cellular model data used to test candidate
    drivers for 'essentiality'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:cc75cad200ac806bfc3926c01e961af1d5a34f471834a05a5facfba517f958e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - DepMap
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rna-essentiality-screen-analysis

## Summary

In-silico validation of iEDGE-predicted cis driver genes by testing their enrichment for amplification-driven gene dependencies from DepMap RNAi (DEMETER2) screens paired with CCLE copy number data, using the iEDGE run_hyperEnrichment one-sided hypergeometric/Fisher framework.

## When to use

Use when a ranked list of candidate cis driver genes (e.g., TCGA Rank-1 iEDGE genes) must be validated against DepMap RNAi essentiality screens where dependency scores correlate with copy number amplification.

## When NOT to use

- When the candidate gene list is already validated against known driver compendia and no functional essentiality evidence is required
- When RNAi dependency data are not matched to copy number data for the same cell lines — the slope test requires paired DepMap/CCLE samples
- When the candidate set derives from trans effects rather than cis (amplification-linked) driver predictions, since the rationale is amplification-driven dependency

## Inputs

- DepMap DEMETER2 RNAi dependency score matrix (genes x cell lines)
- CCLE copy number matrix (genes x cell lines), matched by cell line
- Rank-1 iEDGE cis driver gene list (e.g., from TCGA breast cancer iEDGE run)
- Total gene universe size (ntotal) and minimum draw size (min.drawsize) parameters for run_hyperEnrichment

## Outputs

- Enrichment p-values for the candidate driver set against amplification-driven dependency gene sets (e.g., one-sided Fisher P = 3.53e-5 for BRCA Rank-1 cis genes)
- List of enriched genes in the significant dependency set (e.g., ERBB2, CCNE1, CCND1, FOXA1, ANKRD17, MCL1)
- CSV of full enrichment results (e.g., depmap_dependency_enrichment.csv)

## How to apply

Pair the DepMap DEMETER2 RNAi dependency score matrix with the CCLE copy number matrix by cell line. For each gene, fit a linear regression of dependency score on copy number and test the slope with a one-sided t-test (alternative: slope < 0, i.e., higher copy number implies stronger dependency), then FDR-correct p-values across genes and retain genes with significant negative copy-number-vs-dependency association. Treat this significant set as the 'categories' in iEDGE's run_hyperEnrichment and the Rank-1 cis driver list as the 'drawn' set, performing a one-sided Fisher exact / hypergeometric test with a defined total gene universe (ntotal) and minimum draw size (min.drawsize). In the TCGA breast cancer case, Rank-1 cis genes were significantly enriched for amplification-driven dependencies (one-sided P = 3.53e-5), with the enriched set containing ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1.

## Related tools

- **iEDGE** (Provides run_hyperEnrichment, the one-sided hypergeometric/Fisher enrichment test used to test the candidate cis driver list against amplification-driven dependency gene sets) — https://github.com/montilab/iEDGE
- **R** (Implementation language for the linear regression slope tests, FDR correction, and enrichment workflow)
- **DepMap** (Source of RNAi genetic screens and cellular model data used to test candidate drivers for 'essentiality')

## Examples

```
res <- run_hyperEnrichment(drawn = rank1_cis_genes, categories = amp_driven_dependency_genes, ntotal = ntotal, min.drawsize = 15, mht = "BH")
```

## Evaluation signals

- One-sided Fisher/hypergeometric enrichment p-value should reproduce P = 3.53e-5 for TCGA breast cancer Rank-1 cis genes vs amplification-driven dependency genes
- Enriched gene set should contain ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1
- Regression slope of dependency score on copy number must be negative for retained genes after FDR correction
- DepMap and CCLE matrices must be joined without cell-line loss (paired samples by cell line)
- Output CSV (depmap_dependency_enrichment.csv) contains full per-gene-set enrichment results with finite p-values

## Limitations

- Side-by-side comparison with competing methods like CONEXIC was not possible because its software is not available
- Further experimental studies are needed to validate and characterize predicted drivers; enrichment is an in-silico validation only
- Assumption that number of mediated trans genes proxies driver importance may not hold for specific use cases

## Evidence

- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [discussion] DepMap essentiality validation: [evidence span withheld — anchor status: not_found]
- [abstract] amplification-driven dependency enrichment finding: "predicted drivers were enriched for functionally relevant cancer genes with amplification-driven dependencies, which are of potential prognostic and therapeutic value"
- [readme] iEDGE availability as R package: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
