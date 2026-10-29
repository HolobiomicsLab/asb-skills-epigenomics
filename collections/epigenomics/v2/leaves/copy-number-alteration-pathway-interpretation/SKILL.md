---
name: copy-number-alteration-pathway-interpretation
description: Use when you have pruned iEDGE cis/trans signature gene lists per SCNA
  and need to identify recurrently enriched MSigDB pathways via hypergeometric testing.
when_to_use_negative:
- Input gene sets have not been pruned by differential expression and mediation analysis
  (e.g., unfiltered SCNA gene lists), since enrichment should be run on significant
  drawn sets only
- The goal is to identify cis driver genes themselves rather than annotate cis/trans
  effects by downstream pathway enrichment
- Sample size or alteration group sizes yield drawn gene sets smaller than min.drawsize
  = 3, making hypergeometric over-representation underpowered
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: iEDGE
  role: Provides read_gmt to load MSigDB .gmt files and run_hyperEnrichment to perform
    FDR-corrected hypergeometric over-representation of the pruned cis/trans gene
    sets
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment for assembling per-alteration enrichment tables into enrichment_results.tsv
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:fc9c5df48a9c58eefbb854459866f0b581326927dc2f5a2351bde923f85e5435
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# copy-number-alteration-pathway-interpretation

## Summary

Annotates significant cis/trans gene expression signatures of SCNAs from iEDGE by hypergeometric over-representation analysis against MSigDB gene sets (h.all, c2.cp, c3.tft), revealing recurrent downstream pathway effects such as HALLMARK_ESTROGEN_RESPONSE_EARLY/LATE and HALLMARK_G2M_CHECKPOINT in TCGA breast cancer.

## When to use

Use when you have pruned iEDGE cis/trans signature gene lists per SCNA and need to identify recurrently enriched MSigDB pathways via hypergeometric testing.

## When NOT to use

- Input gene sets have not been pruned by differential expression and mediation analysis (e.g., unfiltered SCNA gene lists), since enrichment should be run on significant drawn sets only
- The goal is to identify cis driver genes themselves rather than annotate cis/trans effects by downstream pathway enrichment
- Sample size or alteration group sizes yield drawn gene sets smaller than min.drawsize = 3, making hypergeometric over-representation underpowered

## Inputs

- iEDGE BRCA results object with pruned significant cis/trans gene sets (FDR cis cutoff 0.25, trans cutoff 0.01)
- MSigDB gene set annotation files in .gmt format: h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt
- Total gene universe size (ntotal)

## Outputs

- Per-alteration hypergeometric enrichment results (pathway, P-value, corrected significance)
- Single combined enrichment_results.tsv table assembled across all alterations, suitable for heatmap display

## How to apply

First extract the significant cis and trans gene sets from the iEDGE results object after pruning at the study's FDR thresholds (fdr.cis.cutoff = 0.25, fdr.trans.cutoff = 0.01). Read the MSigDB annotation files (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) with iEDGE's read_gmt function. For each drawn gene set per alteration, run iEDGE's run_hyperEnrichment hypergeometric over-representation test with multiple-testing correction, using min.drawsize = 3 and hyperthres = 0.25 to call significantly enriched pathways. The rationale is that recurrent enrichment of hallmark sets (e.g., hits in >75% of SCNAs) reveals global downstream effects of co-occurring copy number alterations beyond the cis loci themselves.

## Related tools

- **iEDGE** (Provides read_gmt to load MSigDB .gmt files and run_hyperEnrichment to perform FDR-corrected hypergeometric over-representation of the pruned cis/trans gene sets) — https://github.com/montilab/iEDGE
- **R** (Runtime environment for assembling per-alteration enrichment tables into enrichment_results.tsv)

## Examples

```
run_hyperEnrichment(drawn, categories, ntotal, min.drawsize = 3, hyperthres = 0.25)
```

## Evaluation signals

- Signature hallmark sets such as HALLMARK_ESTROGEN_RESPONSE_EARLY, HALLMARK_ESTROGEN_RESPONSE_LATE, and HALLMARK_G2M_CHECKPOINT should appear as significant hits in >75% of breast cancer SCNAs
- Every called enriched pathway must have a drawn gene set size >= min.drawsize = 3 and pass hyperthres = 0.25 after multiple-testing correction
- The combined enrichment_results.tsv must contain one row per (alteration, gene set) pair with corrected P-values, suitable for heatmap display
- Running the same hypergeometric test machinery as a one-sided Fisher exact test (as done for driver database enrichment) should yield consistent overlap statistics on the same gene lists

## Limitations

- The Sobel-mediated cis/trans connections underlying the drawn sets are only reliable under adequate sample size and strong correlation effects, which may not hold in all datasets
- Recurrent hallmark enrichment reflects global downstream effects of co-occurring alterations and does not by itself prove direct regulation by the SCNA's cis driver
- The enrichment is only as good as the MSigDB v5.0 category annotations (hallmark, c2.cp, c3.tft) supplied via .gmt files

## Evidence

- [abstract] cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks: "cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks"
- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [methods] MSigDB gmt files loaded: "gs.names <- c("h.all.v5.0.symbols.gmt","c2.cp.v5.0.symbols.gmt", "c3.tft.v5.0.symbols.gmt")"
- [readme] README module 2: "pathway enrichment analysis of significant cis and trans genes associated with the epi-DNA alterations"
