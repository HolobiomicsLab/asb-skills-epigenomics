---
name: oncogene-tumor-suppressor-classification
description: Use when validating a ranked list of predicted cis driver genes (e.g.,
  iEDGE Rank-1 genes from amplifications/deletions) against curated cancer driver
  compendia to confirm that top-ranked genes are enriched for known oncogenes or tumor
  suppressors.
when_to_use_negative:
- The candidate driver list has no alteration-direction information (amplification
  vs deletion), so OG/TN labels cannot be assigned to reference genes
- The goal is pathway-level annotation of cis/trans effects, which is handled by iEDGE's
  run_hyperEnrichment pathway enrichment module rather than driver-database overlap
  testing
- Validation target is functional essentiality against genetic screens (DepMap), not
  enrichment in curated driver compendia
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing the iEDGE results object, Rank-1 cis gene rankings, and
    run_hyperEnrichment hypergeometric enrichment machinery used to frame the Fisher
    exact overlap tests
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Statistical environment in which the one-sided Fisher exact tests of Rank-1
    vs non-Rank-1 driver overlap are computed
- name: rmarkdown
  role: Notebook output format used to run and report the iEDGE analysis workflow
    in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:0a0ec89d048dcb2636cabc0f94b2bfee1e9b5e2b041906830d517cc123f0269d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# oncogene-tumor-suppressor-classification

## Summary

Validates iEDGE-predicted cis drivers by classifying reference driver genes by alteration direction — amplified genes as candidate oncogenes (OG), deleted genes as candidate tumor suppressors (TN) — and testing whether Rank-1 cis genes are enriched for known drivers from TUSON, OMIM, COSMIC CGC, and UniProt using one-sided Fisher exact tests.

## When to use

Use when validating a ranked list of predicted cis driver genes (e.g., iEDGE Rank-1 genes from amplifications/deletions) against curated cancer driver compendia to confirm that top-ranked genes are enriched for known oncogenes or tumor suppressors.

## When NOT to use

- The candidate driver list has no alteration-direction information (amplification vs deletion), so OG/TN labels cannot be assigned to reference genes
- The goal is pathway-level annotation of cis/trans effects, which is handled by iEDGE's run_hyperEnrichment pathway enrichment module rather than driver-database overlap testing
- Validation target is functional essentiality against genetic screens (DepMap), not enrichment in curated driver compendia

## Inputs

- iEDGE results object in R containing mediation (bipartite) output (res$pruning$sig, res$de$cis$full)
- Ranked cis gene lists per copy number alteration with Rank-1 / non-Rank-1 labels
- Curated driver gene sets from TUSON, OMIM, COSMIC Cancer Gene Census, and UniProt
- Alteration direction labels (amplification vs deletion) for each gene

## Outputs

- Table of one-sided Fisher exact test statistics and P-values for each database-by-direction combination (e.g., Supplemental Table S5)
- Binary OG/TN classification of reference driver genes by alteration direction
- Per-database overlap counts of Rank-1 vs non-Rank-1 cis genes with driver sets

## How to apply

Partition cis genes into Rank-1 genes (those mediating the largest number of trans genes per alteration, extracted via res$pruning$sig and res$de$cis$full from the iEDGE results object) and non-Rank-1 genes for each copy number alteration. Compile reference driver gene sets from TUSON, OMIM, COSMIC Cancer Gene Census, and UniProt, labeling amplified genes as candidate oncogenes (OG) and deleted genes as candidate tumor suppressors (TN). For each database and each direction (amplified, deleted, combined), perform a one-sided Fisher exact test of the overlap of Rank-1 cis genes with the reference driver set versus non-Rank-1 cis genes, using the iEDGE/R environment consistent with the package's run_hyperEnrichment hypergeometric machinery. Assemble test statistics and P-values for all database-by-direction combinations into a single table (Supplemental Table S5 in the paper). The rationale is the paper's hypothesis that SCNA-related drivers mediate a larger proportion of downstream trans expression effects, so true drivers should concentrate in the Rank-1 stratum.

## Related tools

- **iEDGE** (R package providing the iEDGE results object, Rank-1 cis gene rankings, and run_hyperEnrichment hypergeometric enrichment machinery used to frame the Fisher exact overlap tests) — https://github.com/montilab/iEDGE
- **R** (Statistical environment in which the one-sided Fisher exact tests of Rank-1 vs non-Rank-1 driver overlap are computed)
- **rmarkdown** (Notebook output format used to run and report the iEDGE analysis workflow in R)

## Examples

```
fisher.test(matrix(c(n_rank1_driver, n_rank1_nondriver, n_nonrank1_driver, n_nonrank1_nondriver), nrow=2), alternative="greater")  # one-sided OG/TN enrichment; cf. iEDGE run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)
```

## Evaluation signals

- Computed one-sided Fisher exact P-values match the reported Supplemental Table S5 values (COSMIC OG P=0.0012, COSMIC TN P=0.002)
- Rank-1 amplified drivers show significant enrichment for known oncogenes in UniProt (P=0.0016), COSMIC (P=0.0012), and the combined union test (P=0.0041)
- The 2x2 contingency tables (Rank-1 vs non-Rank-1 by known-driver status) reconstruct from res$pruning$sig and res$de$cis$full with consistent row/column totals
- All database-by-direction combinations (TUSON, OMIM, COSMIC CGC, UniProt x amplified/deleted/combined) are represented in the assembled table with no missing cells

## Limitations

- Ranking cis genes by number of mediated trans genes assumes mediated trans gene count proxies driver importance, which may not hold for specific use cases
- Sobel mediation test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets, which affects which genes reach Rank-1 status
- Further experimental studies are needed to validate and characterize predicted drivers beyond database enrichment

## Evidence

- [abstract] cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors: "cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors"
- [discussion] first testing for their enrichment with known drivers from multiple cancer driver databases: "first testing for their enrichment with known drivers from multiple cancer driver databases"
- [supplementary] Compile the reference driver gene sets from TUSON, OMIM, COSMIC Cancer Gene Census, and UniProt, labeling amplified genes as candidate oncogenes (OG) and deleted genes as candidate tumor suppressors (TN).: [evidence span withheld — anchor status: not_found]
- [supplementary] perform a one-sided Fisher exact test of the overlap of Rank-1 cis genes with the reference driver set versus non-Rank-1 cis genes: [evidence span withheld — anchor status: not_found]
- [supplementary] Predicted Rank-1 driver genes in amplifications were significantly enriched for known oncogenes in UNIPROT (P-value = 0.0016), COSMIC (P-value = 0.0012): "Predicted Rank-1 driver genes in amplifications were significantly enriched for known oncogenes in UNIPROT (P-value = 0.0016), COSMIC (P-value = 0.0012)"
