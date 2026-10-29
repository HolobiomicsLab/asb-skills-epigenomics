---
name: fisher-exact-enrichment-testing
description: Use when validating that a ranked list of predicted cis driver genes
  (e.g., Rank-1 mediators from iEDGE mediation output) is significantly enriched for
  known driver genes from curated compendia such as COSMIC CGC, UniProt, OMIM, or
  TUSON.
when_to_use_negative:
- There is no ranked cis gene list (e.g., no iEDGE mediation output) and only an unranked
  gene set — a plain enrichment test would not test the Rank-1 vs non-Rank-1 partition
- The copy number alterations were not classified by direction (amplification vs deletion),
  since OG/TN labeling of amplified/deleted genes is required
- The goal is pathway-level annotation of cis/trans genes via GMT files (h.all.v5.0.symbols.gmt,
  c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) rather than driver-database validation
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: iEDGE
  role: R package whose ranked cis driver output (res$pruning$sig, res$de$cis$full)
    and run_hyperEnrichment hypergeometric machinery provide the inputs and statistical
    framework for the Fisher exact tests
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Statistical environment in which the one-sided Fisher exact tests are computed
- name: rmarkdown
  role: 'Reporting format for the iEDGE analysis workflow (output: rmarkdown::html_notebook)'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:d755fc65b3118af1c03170b99ead195eb9bd99ff9dd3f9968ed02b4e2677805f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fisher-exact-enrichment-testing

## Summary

In silico validation of iEDGE-predicted Rank-1 cis driver genes by one-sided Fisher exact tests of their overlap with curated cancer driver sets (TUSON, OMIM, COSMIC CGC, UniProt), run separately for amplified (oncogene, OG) and deleted (tumor suppressor, TN) directions.

## When to use

Use when validating that a ranked list of predicted cis driver genes (e.g., Rank-1 mediators from iEDGE mediation output) is significantly enriched for known driver genes from curated compendia such as COSMIC CGC, UniProt, OMIM, or TUSON.

## When NOT to use

- There is no ranked cis gene list (e.g., no iEDGE mediation output) and only an unranked gene set — a plain enrichment test would not test the Rank-1 vs non-Rank-1 partition
- The copy number alterations were not classified by direction (amplification vs deletion), since OG/TN labeling of amplified/deleted genes is required
- The goal is pathway-level annotation of cis/trans genes via GMT files (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) rather than driver-database validation

## Inputs

- iEDGE results object (res$pruning$sig significant cn-cis-trans connections; res$de$cis$full cis DE table)
- Rank-1 cis gene lists per copy number alteration from the mediation (bipartite) output
- Non-Rank-1 cis gene lists per alteration
- Reference driver gene sets: TUSON, OMIM, COSMIC Cancer Gene Census, UniProt (OG labels for amplified genes, TN labels for deleted genes)

## Outputs

- Table of test statistics and one-sided Fisher exact P-values for all database-by-direction (amplified, deleted, combined) combinations
- Binary overlap contingency tables of Rank-1 vs non-Rank-1 cis genes against each driver compendium

## How to apply

Load the iEDGE results object in R and extract ranked cis gene lists from the mediation (bipartite) output, using res$pruning$sig for significant cn-cis-trans connections and res$de$cis$full for the full cis differential expression table. Partition cis genes into Rank-1 genes (those mediating the largest number of trans genes per alteration, per the hypothesis that drivers mediate a larger proportion of downstream trans effects) and non-Rank-1 genes for each copy number alteration. Compile reference driver gene sets, labeling amplified genes as candidate oncogenes (OG) and deleted genes as candidate tumor suppressors (TN). For each database and direction (amplified, deleted, combined union), perform a one-sided Fisher exact test of the overlap of Rank-1 cis genes with the reference set versus non-Rank-1 cis genes, consistent with iEDGE's run_hyperEnrichment hypergeometric enrichment machinery, then assemble all database-by-direction statistics into a single table.

## Related tools

- **iEDGE** (R package whose ranked cis driver output (res$pruning$sig, res$de$cis$full) and run_hyperEnrichment hypergeometric machinery provide the inputs and statistical framework for the Fisher exact tests) — https://github.com/montilab/iEDGE
- **R** (Statistical environment in which the one-sided Fisher exact tests are computed)
- **rmarkdown** (Reporting format for the iEDGE analysis workflow (output: rmarkdown::html_notebook))

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- Computed one-sided Fisher exact P-values reproduce Supplemental Table S5 values: COSMIC OG P=0.0012 and COSMIC TN P=0.002
- Rank-1 amplified drivers show significant enrichment (P<0.01) for UniProt (P=0.0016), COSMIC (P=0.0012), and the combined union test (P=0.0041)
- The enrichment is direction-specific: amplified genes are tested against OG sets and deleted genes against TN sets, with no cross-direction pooling
- The same Rank-1/non-Rank-1 gene partition is used across all database-by-direction tests so P-values are comparable
- All database-by-direction combinations (TUSON, OMIM, COSMIC, UniProt; amplified, deleted, combined) appear in the assembled results table

## Limitations

- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- The Sobel mediation test that produces the Rank-1 rankings is valid only under adequate sample size and strong correlation effects, conditions not met in all datasets
- Further experimental studies are needed to validate and characterize predicted drivers beyond database enrichment

## Evidence

- [abstract] cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors: "cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors"
- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [discussion] In silico validation via driver database enrichment: [evidence span withheld — anchor status: not_found]
- [readme] iEDGE package description: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA"
