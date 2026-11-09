---
name: gene-identifier-mapping-for-enrichment
description: Use when BIT output rank tables (e.g., mm10_RUNX1_rank_table.csv) contain
  TR gene symbols but the downstream enrichment tool requires Entrez IDs, such as
  clusterProfiler::enrichGO.
when_to_use_negative:
- Input gene list already contains Entrez IDs compatible with the OrgDb — remapping
  from symbols would be redundant and risk losing TRs
- The TR identifiers are not standard gene symbols and cannot be reliably matched
  via SYMBOL-type mapping (e.g., ambiguous aliases or protein identifiers)
- The goal is TR identification itself (BIT ranking) rather than downstream GO enrichment
  of an already-ranked TR set
edam_operation: http://edamontology.org/operation_3680
tools:
- name: clusterProfiler
  role: Provides the gene identifier mapping (SYMBOL to Entrez) and over-representation
    analysis functions (enrichGO) that consume the mapped TR list
- name: BIT
  role: Produces the ranked TR output tables whose top-ranked gene symbols are the
    input to the identifier mapping
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Environment in which the mapping (bitr/OrgDb) and enrichment (clusterProfiler)
    are executed
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:4dae9f97186dfbdbbe93fa00f27217ab293f85e9652b951029af06753e79c47a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BIT
  - R
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_3680
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gene-identifier-mapping-for-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map BIT top-ranked TR gene symbols to Entrez gene identifiers so clusterProfiler's over-representation functions (enrichGO) can run GO enrichment on the ranked TR sets from differential accessibility analyses.

## When to use

Use when BIT output rank tables (e.g., mm10_RUNX1_rank_table.csv) contain TR gene symbols but the downstream enrichment tool requires Entrez IDs, such as clusterProfiler::enrichGO.

## When NOT to use

- Input gene list already contains Entrez IDs compatible with the OrgDb — remapping from symbols would be redundant and risk losing TRs
- The TR identifiers are not standard gene symbols and cannot be reliably matched via SYMBOL-type mapping (e.g., ambiguous aliases or protein identifiers)
- The goal is TR identification itself (BIT ranking) rather than downstream GO enrichment of an already-ranked TR set

## Inputs

- BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) with top-ranked TR gene symbols (top 10 for perturbation experiments, top 20 otherwise)
- Organism annotation database: org.Mm.eg.db (mm10 datasets) or org.Hs.eg.db (hg38 datasets)

## Outputs

- Entrez gene identifier list corresponding to the top-ranked TR symbols, formatted for clusterProfiler over-representation functions
- Ranked GO enrichment table written to GO_enrichment_results.csv with GO term IDs, adjusted p-value and q-value columns

## How to apply

Extract the top-ranked TR gene symbols from the BIT output rank table — top 10 for perturbation experiments (e.g., RUNX1 knockout, K562 CRISPR perturbations) and top 20 for cancer-type and cell-type-specific cases. Use clusterProfiler's identifier mapping (e.g., bitr with keyType = 'SYMBOL') against the organism-appropriate OrgDb — org.Mm.eg.db for mm10 datasets, org.Hs.eg.db for hg38 datasets — to convert symbols to Entrez gene identifiers. Only symbols with a successful mapping enter the enrichment gene list; unmapped symbols are dropped. The mapped Entrez IDs are then passed to clusterProfiler::enrichGO with pAdjustMethod = 'BH' and qvalueCutoff = 0.05, and enrichment results are ranked by Benjamini-Hochberg adjusted p values.

## Related tools

- **clusterProfiler** (Provides the gene identifier mapping (SYMBOL to Entrez) and over-representation analysis functions (enrichGO) that consume the mapped TR list)
- **BIT** (Produces the ranked TR output tables whose top-ranked gene symbols are the input to the identifier mapping) — https://github.com/ZeyuL01/BIT
- **R** (Environment in which the mapping (bitr/OrgDb) and enrichment (clusterProfiler) are executed) — https://github.com/ZeyuL01/BIT

## Examples

```
Rscript -e 'ids <- clusterProfiler::bitr(top_tr_symbols, keyType = "SYMBOL", toType = "ENTREZID", OrgDb = org.Mm.eg.db); ego <- clusterProfiler::enrichGO(gene = ids$ENTREZID, OrgDb = org.Mm.eg.db, pAdjustMethod = "BH", qvalueCutoff = 0.05)'
```

## Evaluation signals

- Output table GO_enrichment_results.csv exists and contains GO term IDs with adjusted p-value and q-value columns
- All reported GO terms have BH-adjusted q-values <= 0.05, matching the qvalueCutoff = 0.05 threshold
- Mapped Entrez IDs resolve correctly against the organism-appropriate OrgDb (org.Mm.eg.db for mm10, org.Hs.eg.db for hg38)
- Number of genes entering enrichGO equals (or is traceably less than, due to unmapped symbols) the intended top-10/top-20 TR set size
- Enriched biological processes reproduce the expected biology: sister chromatid cohesion for the CTCF depletion experiment, erythrocyte differentiation and homeostasis for the ZBTB7A knockout experiment

## Limitations

- Symbols without an Entrez mapping in the OrgDb are silently lost from the enrichment gene list, shrinking the effective TR set
- BIT's performance depends on NGS data quality; sequencing biases such as GC content bias or PCR amplification bias can reduce the reliability of the input epigenomic region sets, and hence of the top-ranked TRs being mapped

## Evidence

- [methods] Map the TR gene symbols to Entrez gene identifiers required by clusterProfiler's over-representation functions.: [evidence span withheld — anchor status: not_found]
