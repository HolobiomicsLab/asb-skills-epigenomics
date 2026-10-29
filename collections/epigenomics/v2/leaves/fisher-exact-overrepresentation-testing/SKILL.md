---
name: fisher-exact-overrepresentation-testing
description: Use when you have a ranked list of TR gene symbols from BIT (e.g., mm10_RUNX1_rank_table.csv)
  and need to test whether the top-ranked TRs are enriched for GO biological processes
  relevant to the perturbed or analyzed biological context.
when_to_use_negative:
- Input is raw sequencing data or peak files (e.g., MACS2-called peaks) rather than
  an already-ranked TR list — BIT must be run first to produce the rank table
- The biological question concerns TR identification itself rather than functional
  interpretation of an existing TR ranking
- The gene list is not from the organism covered by the selected org.* annotation
  database (mm10 datasets require org.Mm.eg.db, hg38 datasets require org.Hs.eg.db)
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: clusterProfiler
  role: Runs the over-representation analysis (enrichGO) with one-sided Fisher's exact
    test, BH adjustment, and qvalueCutoff = 0.05
- name: R
  role: Execution environment; BIT's R package and the enrichment workflow run in
    R
  repo: https://github.com/ZeyuL01/BIT
- name: BIT
  role: Upstream tool producing the ranked TR output lists whose top entries are tested
    for GO enrichment
  repo: https://github.com/ZeyuL01/BIT
- name: org.Mm.eg.db
  role: Mouse Entrez gene annotation background for mm10 datasets
- name: org.Hs.eg.db
  role: Human Entrez gene annotation background for hg38 datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:c17a55a7e38b3aa1e1295a24ed8c795f7fc730d7f8df735545b1ee58cdf2730b
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
  - org.Hs.eg.db
  - org.Mm.eg.db
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fisher-exact-overrepresentation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Over-representation analysis of top-ranked transcriptional regulators (TRs) from BIT output rank tables against Gene Ontology biological process terms, using clusterProfiler's one-sided Fisher's exact test with Benjamini-Hochberg adjustment to test whether the top TRs are enriched for processes relevant to the perturbed biological context.

## When to use

Use when you have a ranked list of TR gene symbols from BIT (e.g., mm10_RUNX1_rank_table.csv) and need to test whether the top-ranked TRs are enriched for GO biological processes relevant to the perturbed or analyzed biological context.

## When NOT to use

- Input is raw sequencing data or peak files (e.g., MACS2-called peaks) rather than an already-ranked TR list — BIT must be run first to produce the rank table
- The biological question concerns TR identification itself rather than functional interpretation of an existing TR ranking
- The gene list is not from the organism covered by the selected org.* annotation database (mm10 datasets require org.Mm.eg.db, hg38 datasets require org.Hs.eg.db)

## Inputs

- BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) with ranked TR gene symbols
- Organism annotation database: org.Mm.eg.db (mm10) or org.Hs.eg.db (hg38)
- Genome build identifier to choose the correct background (mm10 or hg38)

## Outputs

- GO_enrichment_results.csv containing GO term IDs with adjusted p-value and q-value columns, ranked by BH-adjusted p values
- Bar/dot plot of the most significant GO biological process terms for each analyzed TR set
- Set of enriched GO terms with BH-adjusted q-values <= 0.05

## How to apply

Load the BIT output rank table and extract the top-ranked TR gene symbols: the top 10 for perturbation experiments (e.g., RUNX1 knockout, K562 CRISPR perturbations such as CTCF depletion and ZBTB7A knockout) and the top 20 for cancer-type and cell-type-specific analyses. Map the gene symbols to Entrez gene identifiers required by clusterProfiler's over-representation functions. Run clusterProfiler::enrichGO using a one-sided Fisher's exact test with pAdjustMethod = 'BH' and qvalueCutoff = 0.05, selecting the organism-appropriate annotation database: org.Mm.eg.db for mm10 datasets and org.Hs.eg.db for hg38 datasets. Rank enriched GO terms by Benjamini-Hochberg adjusted p values, retain only terms passing the q-value 0.05 threshold, write the ranked table to GO_enrichment_results.csv, and generate a summary bar/dot plot of the most significant GO terms per TR set. In this study this workflow revealed sister chromatid cohesion formation as the top enriched process for the CTCF depletion experiment and erythrocyte differentiation/homeostasis for the ZBTB7A knockout experiment.

## Related tools

- **clusterProfiler** (Runs the over-representation analysis (enrichGO) with one-sided Fisher's exact test, BH adjustment, and qvalueCutoff = 0.05)
- **R** (Execution environment; BIT's R package and the enrichment workflow run in R) — https://github.com/ZeyuL01/BIT
- **BIT** (Upstream tool producing the ranked TR output lists whose top entries are tested for GO enrichment) — https://github.com/ZeyuL01/BIT
- **org.Mm.eg.db** (Mouse Entrez gene annotation background for mm10 datasets)
- **org.Hs.eg.db** (Human Entrez gene annotation background for hg38 datasets)

## Examples

```
ego <- enrichGO(gene = bit_top_entrez_ids, OrgDb = org.Mm.eg.db, keyType = "ENTREZID", pAdjustMethod = "BH", qvalueCutoff = 0.05)
```

## Evaluation signals

- The output file GO_enrichment_results.csv exists and contains GO term IDs with adjusted p-value and q-value columns
- All reported enriched terms have Benjamini-Hochberg adjusted q-values <= 0.05, matching the analysis cutoff
- TR gene symbols were successfully converted to Entrez gene identifiers before calling enrichGO (no unmapped IDs silently dropped)
- Correct organism background was used: org.Mm.eg.db for mm10 datasets and org.Hs.eg.db for hg38 datasets
- Recovered biological processes match expected biology: e.g., sister chromatid cohesion formation for CTCF depletion and erythrocyte differentiation/homeostasis for ZBTB7A knockout

## Limitations

- BIT's performance can be reduced by biological complexities of TR regulation, such as dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding among TRs, which can propagate into the enrichment results
- Sequencing biases (e.g., GC content bias on the Illumina platform and PCR amplification bias) can reduce the reliability of the input epigenomic region sets and hence the ranked TR lists feeding the enrichment test
- The number of top TRs tested depends on analysis type (top 10 for perturbation experiments, top 20 for other analyses), and small lists limit statistical power of the Fisher's exact test

## Evidence
