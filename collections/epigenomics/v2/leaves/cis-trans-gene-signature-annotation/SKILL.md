---
name: cis-trans-gene-signature-annotation
description: Use when significant cis/trans gene sets from iEDGE differential expression
  and mediation analysis need functional annotation, e.g. detecting hallmark pathways
  recurrently enriched across multiple SCNAs.
when_to_use_negative:
- Input is the raw iEDGE output before cis/trans gene sets have been pruned at the
  specified FDR cutoffs — enrichment should be run on drawn (significant) gene sets
  only
- Gene sets are methylation or other non-expression signatures rather than the cis/trans
  expression signatures the iEDGE pipeline defines
- A ranked-list / GSEA-style test is desired rather than a hypergeometric over-representation
  test on discrete gene sets with min.drawsize filtering
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: R package providing read_gmt and run_hyperEnrichment to perform pathway enrichment
    analysis of significant cis and trans genes associated with epi-DNA alterations
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment in which the iEDGE enrichment workflow is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:8195a3c92f2612f552a21c2fc8fc25f62d10146009d4cafe9021744e321c6f3e
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cis-trans-gene-signature-annotation

## Summary

Annotates iEDGE cis and trans gene expression signatures associated with (epi-)DNA alterations by running a hypergeometric over-representation test against MSigDB gene sets, revealing recurrent downstream pathways of candidate driver alterations.

## When to use

Use when significant cis/trans gene sets from iEDGE differential expression and mediation analysis need functional annotation, e.g. detecting hallmark pathways recurrently enriched across multiple SCNAs.

## When NOT to use

- Input is the raw iEDGE output before cis/trans gene sets have been pruned at the specified FDR cutoffs — enrichment should be run on drawn (significant) gene sets only
- Gene sets are methylation or other non-expression signatures rather than the cis/trans expression signatures the iEDGE pipeline defines
- A ranked-list / GSEA-style test is desired rather than a hypergeometric over-representation test on discrete gene sets with min.drawsize filtering

## Inputs

- iEDGE results object with significant cis and trans gene sets (pruned at fdr.cis.cutoff and fdr.trans.cutoff)
- MSigDB GMT annotation files: h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt
- Total universe gene count (ntotal) for the hypergeometric background

## Outputs

- Significant pathway enrichment results per alteration (hypergeometric p-values with multiple-testing correction)
- Assembled enrichment table for heatmap display, e.g. enrichment_results.tsv

## How to apply

Load the iEDGE results object and extract significant cis and trans gene sets pruned at FDR thresholds (e.g. fdr.cis.cutoff = 0.25, fdr.trans.cutoff = 0.01). Read MSigDB annotation files (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) with the iEDGE read_gmt function. Run run_hyperEnrichment, a hypergeometric over-representation test with multiple-testing correction, using min.drawsize = 3 and hyperthres = 0.25 to call significantly enriched pathways. Assemble significant enrichments across all alterations into a single table (enrichment_results.tsv) suitable for heatmap display to identify pathways such as HALLMARK_ESTROGEN_RESPONSE_LATE, HALLMARK_ESTROGEN_RESPONSE_EARLY, and HALLMARK_G2M_CHECKPOINT that are hits in >75% of breast cancer SCNAs.

## Related tools

- **iEDGE** (R package providing read_gmt and run_hyperEnrichment to perform pathway enrichment analysis of significant cis and trans genes associated with epi-DNA alterations) — https://github.com/montilab/iEDGE
- **R** (Runtime environment in which the iEDGE enrichment workflow is executed)

## Examples

```
run_hyperEnrichment(drawn, categories, ntotal, min.drawsize = 3, mht, verbose = TRUE, order, hypercol, hyperthres = 0.25)
```

## Evaluation signals

- Every enriched pathway reported has at least min.drawsize = 3 drawn genes from the significant cis/trans signature
- Only pathways passing the hyperthres = 0.25 threshold (with multiple-testing correction) appear in the results table
- GMT file names parsed match the expected MSigDB collections (h.all, c2.cp, c3.tft v5.0 symbols)
- Output table contains enrichment results for all analyzed alterations and is schema-valid for heatmap rendering (e.g. enrichment_results.tsv)
- In the BRCA reference application, recurrent hallmarks such as HALLMARK_ESTROGEN_RESPONSE_LATE, HALLMARK_ESTROGEN_RESPONSE_EARLY, and HALLMARK_G2M_CHECKPOINT are significant in >75% of SCNAs

## Limitations

- Hypergeometric enrichment depends on the drawn gene sets already being correctly pruned at the chosen FDR cutoffs; mis-set thresholds change the enrichment results
- The min.drawsize = 3 filter removes very small signatures from enrichment testing
- Signatures must be gene symbols compatible with the v5.0 MSigDB symbol-based GMT files

## Evidence

- [abstract] cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks: "cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks"
- [methods] run_hyperEnrichment function signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [methods] GMT gene set annotation files used: "gs.names <- c("h.all.v5.0.symbols.gmt","c2.cp.v5.0.symbols.gmt", "c3.tft.v5.0.symbols.gmt")"
- [readme] iEDGE package module description: "pathway enrichment analysis of significant cis and trans genes associated with the epi-DNA alterations"
