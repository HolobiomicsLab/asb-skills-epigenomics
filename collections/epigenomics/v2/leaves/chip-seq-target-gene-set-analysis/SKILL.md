---
name: chip-seq-target-gene-set-analysis
description: Use when you have a BED file of TF ChIP-seq peaks and want to learn the
  biological function of the target genes linked to those binding regions via GO/KEGG
  enrichment.
when_to_use_negative:
- The goal is to identify enriched TF motifs within the REM sequences rather than
  function of associated genes — use PASTAA motif enrichment instead
- You need REM–gene associations de novo from chromatin accessibility/expression data
  rather than linking existing ChIP-seq peaks to genes — use EpiRegio Gene Query/StitchIt
  workflow directly
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: EpiRegio
  role: Region Query of the TF ChIP-seq peaks against REMs with 50% overlap to produce
    the associated gene list used as enrichment input
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: g:Profiler
  role: GO term and KEGG pathway enrichment analysis (default parameters, Benjamini–Hochberg
    FDR 0.05) of the ARID3A-associated gene IDs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:d93309227f95ab206ecdbba1a57d7a5181e03b361b7d17722cfc3c7fad6b896e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - g:Profiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chip-seq-target-gene-set-analysis

## Summary

Functional enrichment analysis (g:Profiler, default parameters, Benjamini–Hochberg FDR 0.05) of the gene set associated with a transcription factor's ChIP-seq binding regions, as demonstrated by mapping ARID3A peaks (ENCFF002CVL) to REMs with EpiRegio and recovering SLE/alcoholism KEGG pathways and chromatin-assembly GO terms.

## When to use

Use when you have a BED file of TF ChIP-seq peaks and want to learn the biological function of the target genes linked to those binding regions via GO/KEGG enrichment.

## When NOT to use

- The goal is to identify enriched TF motifs within the REM sequences rather than function of associated genes — use PASTAA motif enrichment instead
- You need REM–gene associations de novo from chromatin accessibility/expression data rather than linking existing ChIP-seq peaks to genes — use EpiRegio Gene Query/StitchIt workflow directly

## Inputs

- TF ChIP-seq peaks BED file from ENCODE (e.g. ENCFF002CVL for ARID3A)
- Gene ID list associated with ChIP-seq peaks from an EpiRegio Region Query (50% overlap), here 1,721 genes (duplicates removed)

## Outputs

- Significant enrichment terms: KEGG pathways (systemic lupus erythematosus, alcoholism) and GO biological processes (chromatin assembly or disassembly, nucleosome organization) at BH FDR <= 0.05
- Enrichment results table exported as gprofiler_results.csv

## How to apply

Download the TF ChIP-seq peaks (e.g. ARID3A accession ENCFF002CVL) from ENCODE as a BED file, then run an EpiRegio Region Query searching for REMs overlapping at least 50% with the peaks to obtain the associated gene IDs. Deduplicate the gene IDs (g:Profiler considers each unique gene only once) and submit the full list of 1,721 genes into the g:Profiler gost input field rather than a web URL, which only fits ~90 genes. Run g:Profiler with default parameters on GO terms and KEGG pathways and declare terms significant at Benjamini–Hochberg adjusted p-value <= 0.05. Extract the significant pathways and top biological processes and export the enrichment table as gprofiler_results.csv.

## Related tools

- **EpiRegio** (Region Query of the TF ChIP-seq peaks against REMs with 50% overlap to produce the associated gene list used as enrichment input) — https://github.com/TeamRegio/EpiRegioDB
- **g:Profiler** (GO term and KEGG pathway enrichment analysis (default parameters, Benjamini–Hochberg FDR 0.05) of the ARID3A-associated gene IDs)

## Examples

```
Rscript workflow.R  # after: bedtools getfasta on REMs / EpiRegio Region Query (50% overlap) -> paste 1721 gene IDs into https://biit.cs.ut.ee/gprofiler/gost
```

## Evaluation signals

- The significant terms at Benjamini–Hochberg FDR <= 0.05 include the KEGG systemic lupus erythematosus and alcoholism pathways, matching the paper's finding
- Top significant GO biological processes include chromatin assembly or disassembly and nucleosome organization
- The gene set passed to g:Profiler contains 1,721 unique gene IDs (duplicates removed, since g:Profiler counts each gene once)
- The Region Query used the 50% minimum overlap percentage between ChIP-seq peaks and REMs
- The enrichment table is exported as gprofiler_results.csv with adjusted p-values available for filtering

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- No tool versions are stated for the software used (PASTAA, EpiRegio, etc.)
- If submitting via a web URL, only the first ~90 unique genes fit in the URL; the full list must be pasted into the g:Profiler gost input field

## Evidence

- [methods] Region Query of ChIP-seq peaks against REMs with 50% overlap: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [methods] GO term enrichment analysis of resulting genes with g:Profiler: "perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs"
- [discussion] ARID3A regulates genes in SLE and alcoholism pathways: [evidence span withheld — anchor status: not_found]
- [methods] Download TF ChIP-seq peaks as BED file: "Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file"
- [readme] EpiRegio is a resource of REMs associated with genes: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
