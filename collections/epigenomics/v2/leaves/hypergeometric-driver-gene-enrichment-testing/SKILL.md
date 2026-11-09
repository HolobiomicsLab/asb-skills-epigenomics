---
name: hypergeometric-driver-gene-enrichment-testing
description: Use when a ranked candidate driver list (e.g., iEDGE Rank-1 cis genes)
  must be tested for significant overlap with known cancer driver databases to validate
  a mediation/differential-expression prioritization.
when_to_use_negative:
- The gene list is a pathway enrichment result or unranked arbitrary gene set with
  fewer than the min.drawsize=3 threshold in a category — the one-sided Fisher test
  will not be interpretable
- No curated driver compendium (validated oncogenes/tumor suppressors) is available
  to serve as the category background, or the analysis goal is differential expression
  rather than driver-list validation
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: iEDGE
  role: Provides run_hyperEnrichment, which applies the one-sided Fisher exact (hypergeometric)
    test with FDR correction to each candidate driver list against driver-database
    compendia
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Implementation environment for the iEDGE package functions and aggregation
    of per-type Fisher statistics with the KS test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:975b3556609bf035c889eab1acc64142604efa49dd7d7d66a4570449b98475b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hypergeometric-driver-gene-enrichment-testing

## Summary

In silico validation of predicted candidate cancer drivers by one-sided Fisher exact (hypergeometric) enrichment of each per-sample driver list against compendia of known oncogenes and tumor suppressors, with FDR correction, as used to validate iEDGE Rank-1 cis drivers across 19 TCGA cancer types.

## When to use

Use when a ranked candidate driver list (e.g., iEDGE Rank-1 cis genes) must be tested for significant overlap with known cancer driver databases to validate a mediation/differential-expression prioritization.

## When NOT to use

- The gene list is a pathway enrichment result or unranked arbitrary gene set with fewer than the min.drawsize=3 threshold in a category — the one-sided Fisher test will not be interpretable
- No curated driver compendium (validated oncogenes/tumor suppressors) is available to serve as the category background, or the analysis goal is differential expression rather than driver-list validation

## Inputs

- Per-cancer-type list of Rank-1 pruned cis candidate driver genes (gene symbols)
- Compendia of known cancer driver genes from multiple driver databases (validated oncogenes and tumor suppressors)
- Total universe gene count (ntotal) for the hypergeometric background
- Optional amplification-dependency driver subset (Rank-1 drivers with amplification-driven dependencies)

## Outputs

- Per-driver-list Fisher exact test statistics and FDR-corrected p-values vs each driver compendium
- Count of cancer types with significant known-driver enrichment (expected 15 of 19)
- Kolmogorov-Smirnov test p-value comparing Rank-1 vs non-Rank-1 enrichment (expected p = 2.5e-13; amplification subset p = 0.00287)

## How to apply

For each cancer type, take the Rank-1 pruned cis candidate drivers (from calc_sobel with bipartite=TRUE and prune.col='fdr', prune.thres=0.05) as the 'drawn' gene set and query each compendium of validated oncogenes and tumor suppressors as the category set. Run run_hyperEnrichment with min.drawsize=3 and hyperthres=0.25, applying the one-sided Fisher exact test with multiple-hypothesis (FDR) correction to test whether known drivers are over-represented. Aggregate the per-type Fisher p-values across all 19 TCGA cancer types and compute a Kolmogorov-Smirnov test comparing Rank-1 versus non-Rank-1 driver enrichment, both overall and for the amplification-driven dependency subset. Enrichment significance (FDR < 0.05) serves as the in silico validation criterion that predicted cis drivers resemble known cancer drivers.

## Related tools

- **iEDGE** (Provides run_hyperEnrichment, which applies the one-sided Fisher exact (hypergeometric) test with FDR correction to each candidate driver list against driver-database compendia) — https://github.com/montilab/iEDGE
- **R** (Implementation environment for the iEDGE package functions and aggregation of per-type Fisher statistics with the KS test)

## Examples

```
run_hyperEnrichment(drawn = rank1_drivers, categories = driver_compendia, ntotal = ntotal, min.drawsize = 3, mht = "BH", hyperthres = 0.25)
```

## Evaluation signals

- Reproduced per-type enrichment (Supplemental Table S7) must show 15/19 TCGA cancer types significantly enriched for known drivers (FDR < 0.05)
- KS test of per-type enrichment p-values against uniform must yield p = 2.5e-13 (8/19 for the amplification-dependency subset, KS p = 0.00287; Supplemental Table S8)
- Each tested driver list must meet min.drawsize=3; categories with hyperthres=0.25 overlap are the reported hits
- FDR correction must be applied across the multiple compendia/tests before calling significance
- output_matches_reference comparison against published Supplemental Tables S7 and S8

## Limitations

- The Sobel mediation ranking that produces the Rank-1 driver lists is valid only with adequate sample size and strong correlation effects, conditions not met in all TCGA datasets
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Enrichment is an in silico validation only; further experimental studies are needed to validate and characterize predicted drivers

## Evidence

- [methods] run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres): "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [discussion] first testing for their enrichment with known drivers from multiple cancer driver databases: "first testing for their enrichment with known drivers from multiple cancer driver databases"
- [abstract] cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors: "cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors"
