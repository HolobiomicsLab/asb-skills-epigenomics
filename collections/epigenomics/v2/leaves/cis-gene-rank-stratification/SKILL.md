---
name: cis-gene-rank-stratification
description: Use when an iEDGE mediation (bipartite/pruning) output for SCNAs yields
  ranked cis gene lists and you must isolate Rank-1 cis genes (most-mediating per
  alteration) from non-Rank-1 genes for downstream driver validation.
when_to_use_negative:
- When the goal is pathway enrichment of significant cis/trans gene sets rather than
  driver-rank stratification and driver-database validation
- When the input lacks a mediation pruning output (e.g., only differential expression
  results without res$pruning$sig), since Rank-1 status is defined by trans gene mediation
  counts
- When comparing iEDGE with tools that only consider genes within the alterations
  (e.g., GISTIC2.0) without a mediation model, since no trans mediation ranking exists
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: R package providing the run_iEDGE mediation/pruning output (res$pruning$sig)
    from which ranked cis gene lists are extracted, and run_hyperEnrichment for hypergeometric
    enrichment
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Statistical environment used to run iEDGE and perform one-sided Fisher exact
    tests of Rank-1 versus non-Rank-1 overlap with driver databases
- name: rmarkdown
  role: Notebook output format for the analysis reporting the stratification and enrichment
    results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:fd160e3dbdf26b817b55ba4b21ced9e2af8a7cbbd8f297b1f78d849dd2a71b5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cis-gene-rank-stratification

## Summary

Partitions iEDGE cis genes into Rank-1 and non-Rank-1 classes per copy number alteration, where Rank-1 genes mediate the largest number of trans genes in the Sobel mediation (pruning) output. This stratification enables in silico validation of predicted drivers via enrichment against curated oncogene/tumor suppressor databases.

## When to use

Use when an iEDGE mediation (bipartite/pruning) output for SCNAs yields ranked cis gene lists and you must isolate Rank-1 cis genes (most-mediating per alteration) from non-Rank-1 genes for downstream driver validation.

## When NOT to use

- When the goal is pathway enrichment of significant cis/trans gene sets rather than driver-rank stratification and driver-database validation
- When the input lacks a mediation pruning output (e.g., only differential expression results without res$pruning$sig), since Rank-1 status is defined by trans gene mediation counts
- When comparing iEDGE with tools that only consider genes within the alterations (e.g., GISTIC2.0) without a mediation model, since no trans mediation ranking exists

## Inputs

- iEDGE results object (R) containing res$pruning$sig (significant cn-cis-trans mediation connections) and res$de$cis$full (full cis DE table)
- Copy number alteration (SCNA) calls, e.g. GISTIC2.0-derived amplified/deleted gene sets
- Reference driver gene sets from TUSON, OMIM, COSMIC Cancer Gene Census, and UniProt (.gmt-style gene lists)

## Outputs

- Per-alteration partition of cis genes into Rank-1 and non-Rank-1 sets
- Table of one-sided Fisher exact test statistics and P-values for each database-by-direction (amplified, deleted, combined) combination

## How to apply

Load the iEDGE results object in R and extract the significant cn-cis-trans connections from res$pruning$sig plus the full cis differential expression table from res$de$cis$full. For each copy number alteration, count the number of trans genes mediated by each cis gene from the Sobel mediation pruning output and designate the cis gene mediating the largest number of trans genes as the Rank-1 gene, leaving all others as non-Rank-1. The rationale is the paper's core hypothesis: SCNA-related drivers of tumorigenesis mediate a larger proportion of the downstream trans gene expression effect than non-drivers. Compile reference driver sets from TUSON, OMIM, COSMIC Cancer Gene Census, and UniProt (amplified genes as candidate oncogenes, deleted genes as candidate tumor suppressors), then compare Rank-1 versus non-Rank-1 cis genes by one-sided Fisher exact (or hypergeometric, per run_hyperEnrichment machinery) tests per database and direction. Assemble all database-by-direction test statistics and P-values into a single table for reporting.

## Related tools

- **iEDGE** (R package providing the run_iEDGE mediation/pruning output (res$pruning$sig) from which ranked cis gene lists are extracted, and run_hyperEnrichment for hypergeometric enrichment) — https://github.com/montilab/iEDGE
- **R** (Statistical environment used to run iEDGE and perform one-sided Fisher exact tests of Rank-1 versus non-Rank-1 overlap with driver databases)
- **rmarkdown** (Notebook output format for the analysis reporting the stratification and enrichment results)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- One-sided Fisher exact P-values reproduce Supplemental Table S5 values: COSMIC OG P=0.0012 and COSMIC TN P=0.002
- Amplified Rank-1 cis genes show significant oncogene enrichment: UNIPROT P=0.0016, COSMIC P=0.0012, combined test (union of databases) P=0.0041
- Every copy number alteration yields exactly one Rank-1 cis gene per direction (largest number of mediated trans genes), with remaining cis genes assigned to non-Rank-1
- Significant cis-trans connections are drawn only from res$pruning$sig, i.e., mediation connections passing the Sobel test, not the full DE table
- Enrichment machinery is consistent with the package's run_hyperEnrichment hypergeometric framework (min.drawsize, multiple-testing correction parameters respected)

## Limitations

- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- Further experimental studies are needed to validate and characterize predicted drivers

## Evidence

- [discussion] the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [intro] SCNA drivers mediate larger downstream effect: "SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers."
- [readme] README describes pruning module: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
- [methods] methods run_iEDGE invocation: "res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".""
- [discussion] enrichment with known drivers: "first testing for their enrichment with known drivers from multiple cancer driver databases"
