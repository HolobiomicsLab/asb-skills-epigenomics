---
name: hypergeometric-pathway-enrichment-analysis
description: Use when you have pruned lists of significant cis/trans gene sets from
  iEDGE differential expression and mediation analysis and need to identify recurrently
  enriched MSigDB pathways across multiple alterations (e.g., TCGA BRCA SCNAs).
when_to_use_negative:
- When no significant cis/trans gene sets have been identified — pruning at FDR thresholds
  must precede enrichment, otherwise the drawn set is meaningless
- When input gene sets are already pathway-level annotations rather than gene lists
  drawn from the tested universe (ntotal)
- When the goal is ranking candidate cis drivers by mediation of trans effects (use
  calc_sobel mediation analysis instead)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: Provides read_gmt to load MSigDB .gmt annotations and run_hyperEnrichment
    to perform the hypergeometric over-representation test with multiple-testing correction
    on cis/trans gene sets
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Execution environment; iEDGE is an open-source R package used in an R/rmarkdown
    notebook workflow
  repo: https://github.com/montilab/iEDGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:8094c3d4944be3141fc0b5754f94cbb3aace1d8da0d932cc115132acbee7c84b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hypergeometric-pathway-enrichment-analysis

## Summary

Annotate iEDGE cis and trans gene expression signatures by hypergeometric over-representation testing of significant gene sets against MSigDB hallmark (h.all), curated (c2.cp), and transcription factor target (c3.tft) gene sets, revealing recurrent downstream regulatory effects of SCNAs.

## When to use

Use when you have pruned lists of significant cis/trans gene sets from iEDGE differential expression and mediation analysis and need to identify recurrently enriched MSigDB pathways across multiple alterations (e.g., TCGA BRCA SCNAs).

## When NOT to use

- When no significant cis/trans gene sets have been identified — pruning at FDR thresholds must precede enrichment, otherwise the drawn set is meaningless
- When input gene sets are already pathway-level annotations rather than gene lists drawn from the tested universe (ntotal)
- When the goal is ranking candidate cis drivers by mediation of trans effects (use calc_sobel mediation analysis instead)

## Inputs

- Significant cis/trans gene set lists extracted from an iEDGE results object, pruned at FDR cutoffs (e.g., fdr.cis.cutoff = 0.25, fdr.trans.cutoff = 0.01)
- MSigDB gene set annotation files in .gmt format: h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt
- Total number of genes tested (ntotal) for the hypergeometric background

## Outputs

- Per-alteration significant pathway enrichment results (gene set, overlap, p-value, multiple-testing-corrected significance)
- Combined enrichment table across all alterations (e.g., enrichment_results.tsv) suitable for heatmap display of recurrent hits

## How to apply

After running iEDGE differential expression and mediation analysis, prune significant cis/trans gene sets at FDR thresholds (e.g., fdr.cis.cutoff = 0.25 and fdr.trans.cutoff = 0.01) per alteration. Read MSigDB annotations with the iEDGE read_gmt function from the .gmt files h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, and c3.tft.v5.0.symbols.gmt. Run run_hyperEnrichment on each drawn gene set — a hypergeometric over-representation test with multiple-testing correction (mht) — using min.drawsize = 3 to exclude tiny gene sets and hyperthres = 0.25 to call significantly enriched pathways. Assemble significant enrichments for all alterations into a single table (e.g., enrichment_results.tsv) suitable for heatmap display of recurrent hits across SCNAs.

## Related tools

- **iEDGE** (Provides read_gmt to load MSigDB .gmt annotations and run_hyperEnrichment to perform the hypergeometric over-representation test with multiple-testing correction on cis/trans gene sets) — https://github.com/montilab/iEDGE
- **R** (Execution environment; iEDGE is an open-source R package used in an R/rmarkdown notebook workflow) — https://github.com/montilab/iEDGE

## Examples

```
hyper <- run_hyperEnrichment(drawn = sig_genesets, categories = read_gmt("h.all.v5.0.symbols.gmt"), ntotal = nrow(gep), min.drawsize = 3, hyperthres = 0.25)
```

## Evaluation signals

- Each enriched pathway call satisfies the hyperthres cutoff (corrected hypergeometric p-value <= 0.25) and involves a drawn gene set of size >= min.drawsize (3)
- Only gene sets loaded via read_gmt from h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, and c3.tft.v5.0.symbols.gmt appear as candidate categories
- Recurrent hallmark hits (e.g., HALLMARK_ESTROGEN_RESPONSE_EARLY/LATE, HALLMARK_G2M_CHECKPOINT) should appear in >75% of BRCA SCNAs in the assembled table
- The combined enrichment table (enrichment_results.tsv) covers every analyzed alteration and is structured for heatmap display

## Limitations

- Significance depends on the FDR pruning cutoffs applied upstream (fdr.cis.cutoff, fdr.trans.cutoff); different cutoffs change the drawn gene set universe
- Small drawn gene sets below min.drawsize = 3 are excluded, so thin signatures cannot be enriched
- hyperthres = 0.25 is a lenient threshold relative to conventional p < 0.05, increasing sensitivity to marginal enrichments

## Evidence

- [abstract] cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks: "cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks"
- [methods] run_hyperEnrichment function signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [methods] gmt gene set files used: "gs.names <- c("h.all.v5.0.symbols.gmt","c2.cp.v5.0.symbols.gmt", "c3.tft.v5.0.symbols.gmt")"
- [readme] README module description: "pathway enrichment analysis of significant cis and trans genes associated with the epi-DNA alterations"
