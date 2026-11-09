---
name: cancer-driver-gene-curation
description: Use when a ranked candidate driver gene list (e.g., iEDGE Rank-1 cis
  genes per SCNA) needs in silico validation against curated oncogene/tumor suppressor
  compendia (COSMIC, TUSON, OMIM, UniProt) or DepMap dependency data via one-sided
  Fisher/hypergeometric enrichment.
when_to_use_negative:
- Predicting drivers de novo from raw SCNA and gene expression matrices — run construct_iEDGE/run_iEDGE
  first; this skill only validates an already-ranked driver list.
- Pathway-level annotation of cis/trans signatures with MSigDB GMTs (h.all.v5.0.symbols.gmt,
  c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) — that is pathway enrichment annotation,
  not driver-compendia validation.
- Reference compendia that cannot be direction-labeled as OG/TN, since the Fisher
  tests stratify Rank-1 overlap by amplified versus deleted alterations.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: iEDGE
  role: Supplies the iEDGE results object (res$pruning$sig, res$de$cis$full) from
    which Rank-1 cis lists are extracted and the run_hyperEnrichment hypergeometric
    enrichment machinery for the one-sided driver tests
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Environment for the one-sided Fisher exact tests and for regressing DepMap
    DEMETER2 dependency scores on CCLE copy number with one-sided t-tests
- name: rmarkdown
  role: 'Documents the enrichment validation workflow as an R Markdown notebook (output:
    rmarkdown::html_notebook)'
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:a0dc4abd94400a38b6d48fd82f3f6453a211cea30ad8a806fa128f97c511e9de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cancer-driver-gene-curation

## Summary

Curates reference cancer driver gene compendia (COSMIC CGC, TUSON, OMIM, UniProt) with direction-of-alteration labels (OG for amplified, TN for deleted) and in silico validates ranked candidate drivers (e.g., iEDGE Rank-1 cis genes) by one-sided Fisher exact / hypergeometric enrichment, optionally extending to DepMap DEMETER2 RNAi dependencies versus CCLE copy number.

## When to use

Use when a ranked candidate driver gene list (e.g., iEDGE Rank-1 cis genes per SCNA) needs in silico validation against curated oncogene/tumor suppressor compendia (COSMIC, TUSON, OMIM, UniProt) or DepMap dependency data via one-sided Fisher/hypergeometric enrichment.

## When NOT to use

- Predicting drivers de novo from raw SCNA and gene expression matrices — run construct_iEDGE/run_iEDGE first; this skill only validates an already-ranked driver list.
- Pathway-level annotation of cis/trans signatures with MSigDB GMTs (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) — that is pathway enrichment annotation, not driver-compendia validation.
- Reference compendia that cannot be direction-labeled as OG/TN, since the Fisher tests stratify Rank-1 overlap by amplified versus deleted alterations.

## Inputs

- iEDGE results object with ranked cis gene lists per copy number alteration (res$pruning$sig, res$de$cis$full), e.g. from the TCGA breast cancer dataset
- Reference driver gene sets: TUSON, OMIM, COSMIC Cancer Gene Census, UniProt
- DepMap DEMETER2 RNAi dependency score matrix
- CCLE copy number matrix paired by cell line

## Outputs

- Single table of one-sided Fisher exact test statistics and P-values for all database-by-direction combinations (amplified/deleted/combined x TUSON/OMIM/COSMIC/UniProt), matching Supplemental Table S5
- depmap_dependency_enrichment.csv with the DepMap amplification-driven dependency enrichment results
- Enriched driver gene set (ERBB2, CCNE1, CCND1, FOXA1, ANKRD17, MCL1)

## How to apply

Extract ranked cis gene lists from the iEDGE results object (res$pruning$sig for significant cn-cis-trans connections, res$de$cis$full for the full cis differential expression table) and partition cis genes per copy number alteration into Rank-1 genes (those mediating the largest number of trans genes) versus non-Rank-1 genes. Compile reference driver gene sets from TUSON, OMIM, COSMIC Cancer Gene Census and UniProt, labeling amplified genes as candidate oncogenes (OG) and deleted genes as candidate tumor suppressors (TN). For each database and direction (amplified, deleted, combined), run a one-sided Fisher exact test of Rank-1 versus non-Rank-1 overlap with the driver set in R via iEDGE's run_hyperEnrichment hypergeometric machinery, assembling all database-by-direction statistics into a single table comparable to Supplemental Table S5. For dependency-based validation, pair DepMap DEMETER2 RNAi dependency scores with CCLE copy number per cell line, regress dependency on copy number with a one-sided t-test (alternative: slope < 0), FDR-correct across genes, and enrich the significant set against the Rank-1 cis list. Reproduction succeeds when P-values match the reported values: COSMIC OG P=0.0012, COSMIC TN P=0.002, UniProt P=0.0016, combined-test P=0.0041, and DepMap dependency enrichment P=3.53e-5 with ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1 in the enriched set.

## Related tools

- **iEDGE** (Supplies the iEDGE results object (res$pruning$sig, res$de$cis$full) from which Rank-1 cis lists are extracted and the run_hyperEnrichment hypergeometric enrichment machinery for the one-sided driver tests) — https://github.com/montilab/iEDGE
- **R** (Environment for the one-sided Fisher exact tests and for regressing DepMap DEMETER2 dependency scores on CCLE copy number with one-sided t-tests)
- **rmarkdown** (Documents the enrichment validation workflow as an R Markdown notebook (output: rmarkdown::html_notebook))

## Examples

```
library(devtools); install_github("montilab/iEDGE"); library(iEDGE)
```

## Evaluation signals

- Computed one-sided Fisher exact P-values match Supplemental Table S5: COSMIC OG P=0.0012 and COSMIC TN P=0.002
- Amplification Rank-1 genes reproduce enrichment for UniProt OG (P=0.0016), COSMIC (P=0.0012) and the combined union test (P=0.0041)
- DepMap enrichment reproduces P=3.53e-5 and the enriched set contains ERBB2, CCNE1, CCND1, FOXA1, ANKRD17 and MCL1
- Output table covers every database-by-direction combination (TUSON/OMIM/COSMIC/UniProt x amplified/deleted/combined) with test statistics
- depmap_dependency_enrichment.csv is written with the full enrichment results

## Limitations

- Rank-1 prioritization relies on the Sobel mediation test, whose validity depends on adequate sample size and strong correlation effects, conditions not met in all TCGA datasets.
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases.
- Enrichment is only in silico validation; further experimental studies are needed to validate and characterize predicted drivers.

## Evidence

- [abstract] iEDGE cis drivers validated by enrichment against known driver compendia: "cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors"
- [discussion] In silico validation strategy: driver database enrichment: "first testing for their enrichment with known drivers from multiple cancer driver databases"
- [discussion] In silico validation strategy: DepMap essentiality: [evidence span withheld — anchor status: not_found]
- [methods] Hypergeometric enrichment function used for driver tests: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [readme] iEDGE package role in the workflow: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
