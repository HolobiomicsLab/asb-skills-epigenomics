---
name: gene-ontology-enrichment-analysis
description: Use when you have a BIT output rank table (e.g., mm10_RUNX1_rank_table.csv)
  of TRs ranked by BIT score from perturbation, cancer-type, or cell-type-specific
  epigenomic query regions and need to test GO BP enrichment of the top candidates.
when_to_use_negative:
- Input is not a ranked TR list from BIT but raw sequencing data (FASTQ, BAM) or peak
  files — those must first be processed through the BIT pipeline (peak calling, differential
  accessibility, BIT scoring)
- You need motif- or sequence-based functional annotation rather than gene-set over-representation
  of known TR gene symbols
- Query regions are from an organism without an appropriate Bioconductor org.*.eg.db
  annotation package
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: clusterProfiler
  role: Runs the over-representation analysis (enrichGO) with one-sided Fisher's exact
    test, BH adjustment, and GO term ranking
- name: R
  role: Language in which BIT's R package and the clusterProfiler enrichment workflow
    are executed
  repo: https://github.com/ZeyuL01/BIT
- name: BIT
  role: Produces the TR rank tables whose top-ranked entries are tested for GO enrichment
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:43d0c52a8cfa19bbca4eed64eedfe5c25531aca8304a64c030e7cb84f6a6425f
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-ontology-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Over-representation analysis of Gene Ontology biological-process terms among top-ranked transcriptional regulators (TRs) from BIT output rank tables, using clusterProfiler's one-sided Fisher's exact test with Benjamini-Hochberg correction to ask whether BIT's top TRs are enriched for processes relevant to the perturbed biological context.

## When to use

Use when you have a BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) of TRs ranked by BIT score from perturbation, cancer-type, or cell-type-specific epigenomic query regions and need to test GO BP enrichment of the top candidates.

## When NOT to use

- Input is not a ranked TR list from BIT but raw sequencing data (FASTQ, BAM) or peak files — those must first be processed through the BIT pipeline (peak calling, differential accessibility, BIT scoring)
- You need motif- or sequence-based functional annotation rather than gene-set over-representation of known TR gene symbols
- Query regions are from an organism without an appropriate Bioconductor org.*.eg.db annotation package

## Inputs

- BIT output rank table CSV of TRs with BIT scores (e.g., mm10_RUNX1_rank_table.csv)
- Gene symbol-to-Entrez ID mappings via org.Mm.eg.db (mm10) or org.Hs.eg.db (hg38)
- Top-N TR list: top 10 genes for perturbation experiments, top 20 for cancer-type and cell-type-specific analyses

## Outputs

- GO_enrichment_results.csv with GO term IDs, adjusted p-values, and q-value columns, ranked by BH-adjusted p-value
- Bar/dot plot of the most significant GO biological process terms per analyzed TR set
- Set of enriched GO terms with BH-adjusted q-values <= 0.05

## How to apply

Load the BIT output rank table and extract top-ranked TR gene symbols: top 10 for perturbation experiments (e.g., RUNX1 knockout, K562 CRISPR perturbations such as CTCF depletion and ZBTB7A knockout) and top 20 for cancer-type and cell-type-specific analyses. Map gene symbols to Entrez IDs required by clusterProfiler's over-representation functions, then run enrichGO with a one-sided Fisher's exact test, pAdjustMethod = 'BH' and qvalueCutoff = 0.05, against the organism database matching the genome build (org.Mm.eg.db for mm10, org.Hs.eg.db for hg38). Rank enriched GO terms by Benjamini-Hochberg adjusted p values and retain only terms with BH-adjusted q-values <= 0.05. Write the ranked table to GO_enrichment_results.csv and generate a bar/dot plot of the most significant terms per TR set.

## Related tools

- **clusterProfiler** (Runs the over-representation analysis (enrichGO) with one-sided Fisher's exact test, BH adjustment, and GO term ranking)
- **R** (Language in which BIT's R package and the clusterProfiler enrichment workflow are executed) — https://github.com/ZeyuL01/BIT
- **BIT** (Produces the TR rank tables whose top-ranked entries are tested for GO enrichment) — https://github.com/ZeyuL01/BIT

## Examples

```
ego <- clusterProfiler::enrichGO(gene = entrez_ids, OrgDb = org.Hs.eg.db, ont = "BP", pAdjustMethod = "BH", qvalueCutoff = 0.05)
```

## Evaluation signals

- Output file GO_enrichment_results.csv exists and contains GO term IDs with adjusted p-value and q-value columns
- All reported GO terms have BH-adjusted q-values <= 0.05 as required by the analysis cutoff
- Enriched terms are biologically coherent with the perturbation context (e.g., sister chromatid cohesion for CTCF depletion; erythrocyte differentiation and homeostasis for ZBTB7A knockout)
- Correct organism background database was used: org.Mm.eg.db for mm10 datasets, org.Hs.eg.db for hg38 datasets
- Correct top-N size was extracted: top 10 TRs for perturbation experiments, top 20 for cancer-type and cell-type-specific cases

## Limitations

- Enrichment results depend on the quality of the upstream BIT ranking; BIT's performance can be reduced by biological complexities of TR regulation such as dynamic TR binding across cell types and cooperative binding
- Interpretation is limited to terms captured in the GO ontology and the chosen organism annotation database
- Results are sensitive to the choice of top-N cutoff (10 vs 20 TRs) taken from the BIT rank table

## Evidence

- [results] Perturbation enrichment findings: [evidence span withheld — anchor status: not_found]
- [readme] BIT package availability in R: [evidence span withheld — anchor status: external_doc]
