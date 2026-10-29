---
name: r-bioinformatics-workflow-scripting
description: Use when you have a BIT output rank table (e.g., mm10_RUNX1_rank_table.csv)
  of TRs ranked from differential accessibility data and need to test whether the
  top TRs are enriched for GO biological processes relevant to the perturbed context.
when_to_use_negative:
- When the input is raw sequencing data rather than a processed BIT rank table — peak
  calling (MACS2 -q 0.01), trimming (trim_galore -q 20 --length 25), and duplicate
  removal (samtools markdup) are upstream steps outside this skill
- When assessing TR binding directly rather than functional enrichment of top-ranked
  TRs — use BIT's scoring/ranking itself or ChIP-seq-based alternatives like BART
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: BIT
  role: R package that produces the TR rank tables (BIT scores from Bayesian hierarchical
    model / MCMC posterior draws) that feed the GO enrichment workflow; repo includes
    manual and manuscript reproduction examples
  repo: https://github.com/ZeyuL01/BIT
- name: clusterProfiler
  role: R package used for over-representation analysis (enrichGO) of top-ranked TRs
    via one-sided Fisher's exact test with BH adjustment
- name: R
  role: The scripting environment in which the BIT package and the GO enrichment analysis
    are executed
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:9f984a6aaa873981e128854cfd01958ff1cb17aad3fbb08367a5cb097b571ace
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# r-bioinformatics-workflow-scripting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An R-based downstream analysis workflow (BIT R package + clusterProfiler) that takes BIT output rank tables, extracts top-ranked transcriptional regulators, and performs GO over-representation analysis to interpret perturbation- or cell-type-specific accessible regions.

## When to use

Use when you have a BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) of TRs ranked from differential accessibility data and need to test whether the top TRs are enriched for GO biological processes relevant to the perturbed context.

## When NOT to use

- When the input is raw sequencing data rather than a processed BIT rank table — peak calling (MACS2 -q 0.01), trimming (trim_galore -q 20 --length 25), and duplicate removal (samtools markdup) are upstream steps outside this skill
- When assessing TR binding directly rather than functional enrichment of top-ranked TRs — use BIT's scoring/ranking itself or ChIP-seq-based alternatives like BART

## Inputs

- BIT output TR rank table CSV (e.g., mm10_RUNX1_rank_table.csv) with ranked TR gene symbols
- Number of top TRs to test (10 for perturbation experiments, 20 for cancer-type/cell-type-specific analyses)
- Organism annotation database (org.Mm.eg.db for mm10, org.Hs.eg.db for hg38)

## Outputs

- GO_enrichment_results.csv with GO term IDs, adjusted p-values, and q-value columns
- Summary bar/dot plot of the most significant GO terms per analyzed TR set

## How to apply

Load the BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) and extract top-ranked TR gene symbols: top 10 for perturbation experiments (RUNX1 knockout, K562 CRISPR perturbations) and top 20 for cancer-type and cell-type-specific analyses. Map TR symbols to Entrez identifiers and run clusterProfiler::enrichGO with a one-sided Fisher's exact test, pAdjustMethod = 'BH' and qvalueCutoff = 0.05, against the organism-appropriate annotation (org.Mm.eg.db for mm10 datasets, org.Hs.eg.db for hg38 datasets). Rank enriched GO terms by Benjamini-Hochberg adjusted p values, retain terms passing the q-value 0.05 cutoff, write the ranked table to GO_enrichment_results.csv, and plot the most significant terms. Rationale: BIT itself is an R package (available on GitHub) whose Bayesian hierarchical model plus MCMC/Gibbs sampling assigns each TR a BIT score, so the R ecosystem is the native environment for both ranking and downstream functional interpretation.

## Related tools

- **BIT** (R package that produces the TR rank tables (BIT scores from Bayesian hierarchical model / MCMC posterior draws) that feed the GO enrichment workflow; repo includes manual and manuscript reproduction examples) — https://github.com/ZeyuL01/BIT
- **clusterProfiler** (R package used for over-representation analysis (enrichGO) of top-ranked TRs via one-sided Fisher's exact test with BH adjustment)
- **R** (The scripting environment in which the BIT package and the GO enrichment analysis are executed) — https://github.com/ZeyuL01/BIT

## Examples

```
ego <- clusterProfiler::enrichGO(gene = bitr(top_trs, fromType='SYMBOL', toType='ENTREZID', OrgDb=org.Mm.eg.db)$ENTREZID, OrgDb='org.Mm.eg.db', keyType='ENTREZID', ont='BP', pAdjustMethod='BH', qvalueCutoff=0.05)
```

## Evaluation signals

- GO_enrichment_results.csv exists and contains GO term IDs with adjusted p-value and q-value columns
- All reported GO terms have BH-adjusted q-values <= 0.05 as required by the analysis cutoff
- The number of tested TRs matches the protocol: top 10 for perturbation experiments, top 20 for remaining analyses
- TR gene symbols were successfully mapped to Entrez IDs compatible with the chosen org.Xx.eg.db database
- Enriched processes match expected biology: sister chromatid cohesion for CTCF depletion, erythrocyte differentiation/homeostasis for ZBTB7A knockout

## Limitations

- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding among TRs
- Sequencing biases such as GC content bias (e.g., Illumina) and PCR amplification bias can reduce the reliability of the input epigenomic region sets feeding the BIT rank tables
- GO enrichment depends on TR ranking quality; comparisons of BIT with other approaches show generally low Jaccard similarity, so top-ranked TR sets may differ across methods

## Evidence

- [other] Load the BIT output rank table (e.g., mm10_RUNX1_rank_table.csv) and extract the top-ranked TR gene symbols: top 10 for perturbation experiments: [evidence span withheld — anchor status: not_found]
