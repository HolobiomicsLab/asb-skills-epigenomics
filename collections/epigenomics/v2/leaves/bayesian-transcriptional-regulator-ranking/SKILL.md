---
name: bayesian-transcriptional-regulator-ranking
description: Use when you have a BED file of differential accessibility or binding
  peak regions from a TR knockout, CRISPR perturbation, cancer-type, or cell-type
  comparison and need to identify/rank the perturbed or cell-type-specific TRs in
  silico with posterior uncertainty.
when_to_use_negative:
- Input is a gene set or expression signature rather than genomic region coordinates
  — BIT requires epigenomic region sets; gene-set-based tools like BART are the alternative
- The query genome is not human or mouse — the pre-compiled reference libraries cover
  only human (hg) and mouse (mm10) TR ChIP-seq data
- You need motif-level inference rather than context-specific binding evidence — BIT
  deliberately bypasses computationally predicted motifs and needs actual peak regions
tools:
- name: BIT
  role: Core Bayesian hierarchical model that ranks TRs from the query region set
    and outputs BIT scores, credible intervals, and ranks
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Language of the BIT R package and of the preprocessing/postprocessing scripts
    (rtracklayer, dplyr, clusterProfiler)
  repo: https://github.com/ZeyuL01/BIT
- name: DiffBind
  role: Produces the differential accessibility report (RUNX1_cKO_report.csv) whose
    Fold column is split into increased/decreased DAR sets
- name: MACS2
  role: Peak caller whose output peak sets (e.g., -q 0.01) serve as the query region
    input format for BIT
- name: clusterProfiler
  role: GO over-representation analysis (enrichGO, one-sided Fisher's exact test,
    BH adjustment, qvalueCutoff 0.05) of top-ranked TRs
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:14c543fdfec969bf9505661ead156b9f0b0397a22671e78aa667a9815296c386
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
  - DiffBind
  - MACS2
  - R
  - clusterProfiler
  - dplyr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bayesian-transcriptional-regulator-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank candidate transcriptional regulators (TRs) from a user-provided epigenomic query region set (e.g., ATAC-seq DARs called by MACS2/DiffBind) using BIT, a Bayesian hierarchical model that measures genome-wide consistency against >10,000 human and >5,000 mouse TR ChIP-seq datasets and quantifies uncertainty with 95% credible intervals.

## When to use

Use when you have a BED file of differential accessibility or binding peak regions from a TR knockout, CRISPR perturbation, cancer-type, or cell-type comparison and need to identify/rank the perturbed or cell-type-specific TRs in silico with posterior uncertainty.

## When NOT to use

- Input is a gene set or expression signature rather than genomic region coordinates — BIT requires epigenomic region sets; gene-set-based tools like BART are the alternative
- The query genome is not human or mouse — the pre-compiled reference libraries cover only human (hg) and mouse (mm10) TR ChIP-seq data
- You need motif-level inference rather than context-specific binding evidence — BIT deliberately bypasses computationally predicted motifs and needs actual peak regions

## Inputs

- Query region set as a BED file of peak chromosome coordinates (e.g., mm10_RUNX1.bed, 15652 ranges; peaks callable by MACS2 or exported from DiffBind)
- DiffBind differential accessibility report (e.g., RUNX1_cKO_report.csv) with a Fold column for splitting into Fold > 0 / Fold < 0 subsets
- Genome build identifier (mm10 or hg38) selecting the pre-compiled reference library (10,140 human TR ChIP-seq datasets / 988 TRs; 5,681 mouse datasets)

## Outputs

- TR rank table CSV (e.g., mm10_RUNX1_rank_table.csv) with columns Theta_i, 95% credible interval lower/upper, BIT_score, BIT_score_lower/upper, and Rank
- Separate rank tables for increased- and decreased-accessibility subsets (RUNX1_KO_increased.bed / RUNX1_KO_decreased.bed inputs)
- Downstream GO enrichment table (GO_enrichment_results.csv) of top-ranked TRs with BH-adjusted p-values and q-values

## How to apply

Convert the query regions to a BED file with chromosome coordinates only (e.g., mm10_RUNX1.bed exported from a DiffBind report via rtracklayer; optionally split with dplyr into Fold > 0 and Fold < 0 subsets to analyze increased vs. decreased accessibility separately). BIT transforms peak coordinates into fixed-length binary vectors, integrates them with its pre-compiled reference library of TR ChIP-seq datasets via a Bayesian hierarchical model, and uses a Gibbs sampler (MCMC) to draw from the joint posterior. Run BIT with the matching genome build (genome="mm10" or hg38) to produce a rank table (e.g., mm10_RUNX1_rank_table.csv) containing Theta_i, lower/upper bounds, BIT_score, BIT_score_lower/upper, and Rank per TR. Acceptance: the perturbed TR should appear in a top position (e.g., Bcl11b Rank with BIT_score 0.10132149, CI 0.09860792-0.10371893, for the full RUNX1 cKO DAR set; Cdk6 0.09366096 for increased; Runx1 0.06684889 for decreased). For interpretation, extract the top 10 TRs (perturbation experiments) or top 20 (cancer/cell-type cases) and run clusterProfiler::enrichGO (one-sided Fisher's exact test, pAdjustMethod='BH', qvalueCutoff=0.05, org.Mm.eg.db/org.Hs.eg.db) on the ranked symbols.

## Related tools

- **BIT** (Core Bayesian hierarchical model that ranks TRs from the query region set and outputs BIT scores, credible intervals, and ranks) — https://github.com/ZeyuL01/BIT
- **R** (Language of the BIT R package and of the preprocessing/postprocessing scripts (rtracklayer, dplyr, clusterProfiler)) — https://github.com/ZeyuL01/BIT
- **DiffBind** (Produces the differential accessibility report (RUNX1_cKO_report.csv) whose Fold column is split into increased/decreased DAR sets)
- **MACS2** (Peak caller whose output peak sets (e.g., -q 0.01) serve as the query region input format for BIT)
- **clusterProfiler** (GO over-representation analysis (enrichGO, one-sided Fisher's exact test, BH adjustment, qvalueCutoff 0.05) of top-ranked TRs)

## Examples

```
positive_fold <- RUNX1_report_tab %>% dplyr::filter(Fold > 0)
```

## Evaluation signals

- Each rank table's Rank 1 row reproduces the reported TR name and BIT_score to 8 decimal places (Bcl11b 0.10132149 full set; Cdk6 0.09366096 increased; Runx1 0.06684889 decreased)
- The reported 95% credible interval brackets the point estimate (e.g., 0.09860792-0.10371893 for Bcl11b)
- Output table schema check: contains Theta_i, lower/upper, BIT_score, BIT_score_lower/upper, and Rank columns
- Perturbed TRs appear in top positions across the full and direction-split DAR analyses (reported as no. 3 and no. 2)
- Downstream GO enrichment: all reported terms have Benjamini-Hochberg adjusted q-values <= 0.05

## Limitations

- BIT can suffer from biological complexities of TR regulation — dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding — which can reduce performance
- Performance depends on NGS data quality; sequencing biases such as GC content bias (Illumina) and PCR amplification bias can reduce reliability of the input region set and binding profiles
- With growing ChIP-seq availability the number of inferred parameters increases significantly; analyzing dozens or hundreds of profiling datasets with the Gibbs sampler can become time-consuming
- Context-specific TR ChIP-seq binding profiles currently cover only a tiny part of all possible experiments, challenging accurate prediction and functional interpretation

## Evidence

- [readme] README describes BIT as an R tool for identifying TRs from user-provided epigenomic regions: [evidence span withheld — anchor status: not_found]
