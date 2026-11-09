---
name: kegg-pathway-enrichment-interpretation
description: Use when you have a deduplicated list of gene IDs linked to a TF's binding
  regions (e.g., from a Region Query of ENCODE ChIP-seq peaks) and need pathway-level
  interpretation such as KEGG disease pathways.
when_to_use_negative:
- Input is a set of DNA regions rather than gene IDs — motif enrichment tools like
  PASTAA on REM sequences are the appropriate choice instead
- You need TF motif enrichment ranked by cell type score (e.g., differentially expressed
  genes in proliferating cells), which EpiRegio addresses with PASTAA rather than
  pathway enrichment
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: g:Profiler
  role: Performs GO term and KEGG pathway enrichment analysis (default parameters)
    on the TF-associated gene list; significant terms are selected at Benjamini–Hochberg
    adjusted p-value <= 0.05.
- name: EpiRegio
  role: Provides the Region Query that maps ENCODE ChIP-seq peaks (e.g., ENCFF002CVL)
    to overlapping REMs and their associated genes, producing the input gene list
    for enrichment.
  repo: https://epiregiodb.readthedocs.io/en/latest/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:3c4c2dfb7e32314d7a29dd76b1b10ce3a969887ab743c723782aca3512579c0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - g:Profiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kegg-pathway-enrichment-interpretation

## Summary

Functional enrichment analysis of a TF's target-gene list (e.g., ARID3A ChIP-seq-associated genes) with g:Profiler to identify significant KEGG pathways and GO biological processes under Benjamini–Hochberg FDR 0.05, as demonstrated in the EpiRegio application scenario.

## When to use

Use when you have a deduplicated list of gene IDs linked to a TF's binding regions (e.g., from a Region Query of ENCODE ChIP-seq peaks) and need pathway-level interpretation such as KEGG disease pathways.

## When NOT to use

- Input is a set of DNA regions rather than gene IDs — motif enrichment tools like PASTAA on REM sequences are the appropriate choice instead
- You need TF motif enrichment ranked by cell type score (e.g., differentially expressed genes in proliferating cells), which EpiRegio addresses with PASTAA rather than pathway enrichment

## Inputs

- Deduplicated list of gene IDs associated with TF ChIP-seq peaks (e.g., 1,721 ARID3A target genes from EpiRegio Region Query)
- BED file of TF ChIP-seq peaks from ENCODE (e.g., ENCFF002CVL) used upstream to derive the gene list

## Outputs

- Enrichment table exported as gprofiler_results.csv with GO terms and KEGG pathways and Benjamini–Hochberg adjusted p-values
- Significant KEGG pathways (e.g., systemic lupus erythematosus, alcoholism) and top GO biological processes (chromatin assembly or disassembly, nucleosome organization) at FDR <= 0.05

## How to apply

Load the gene IDs associated with the TF ChIP-seq peaks (e.g., 1,721 ARID3A-associated genes from an EpiRegio Region Query of ENCODE peak ENCFF002CVL with 50% REM overlap) and deduplicate them, since g:Profiler considers duplicate gene IDs only once; if submitting via web URL only the first 90 unique genes fit, so paste the full list into the g:Profiler gost input field (https://biit.cs.ut.ee/gprofiler/gost). Run g:Profiler with default parameters on all gene IDs to perform GO term and KEGG pathway enrichment. Apply a Benjamini–Hochberg adjusted p-value threshold of 0.05 to declare terms significant, then extract the significant KEGG pathways and top GO biological processes and export the enrichment table as gprofiler_results.csv.

## Related tools

- **g:Profiler** (Performs GO term and KEGG pathway enrichment analysis (default parameters) on the TF-associated gene list; significant terms are selected at Benjamini–Hochberg adjusted p-value <= 0.05.)
- **EpiRegio** (Provides the Region Query that maps ENCODE ChIP-seq peaks (e.g., ENCFF002CVL) to overlapping REMs and their associated genes, producing the input gene list for enrichment.) — https://epiregiodb.readthedocs.io/en/latest/index.html

## Examples

```
gost <- gprofiler2::gost(query = readLines('arid3a_genes.txt'), organism = 'hsapiens'); write.csv(gost$result, 'gprofiler_results.csv')
```

## Evaluation signals

- The significant terms at Benjamini–Hochberg FDR <= 0.05 must include the KEGG systemic lupus erythematosus and alcoholism pathways, matching the enrichments reported in the paper.
- The significant terms must include the GO biological processes chromatin assembly or disassembly and nucleosome organization.
- The input gene list must be deduplicated (1,721 unique ARID3A-associated genes), since g:Profiler considers duplicate gene IDs only once.
- The enrichment table is exported as gprofiler_results.csv with adjusted p-values available for thresholding.

## Limitations

- Only the first 90 unique genes fit in a g:Profiler web URL; the full list must be pasted into the gost input field instead.
- The linear model used in STITCHIT, which underlies the EpiRegio Cell type score of the upstream REMs, is a simplification of actual gene regulation by REMs.
- No tool versions are stated for the software used (g:Profiler, EpiRegio, PASTAA).

## Evidence

- [methods] perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs: "perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs"
- [discussion] we were able to recapture findings from previous studies by showing that ARID3A is involved in regulating a network of genes found in pathways of SLE and alcoholism: [evidence span withheld — anchor status: not_found]
- [methods] we use the ChIP-seq peaks of TF ARID3A with the accession number ENCFF002CVL: "we use the ChIP-seq peaks of TF ARID3A with the accession number ENCFF002CVL"
- [methods] search for REMs overlapping at least 50% with the TF ChIP-seq peaks: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [readme] This repository contains examples and code to perform the Application scenarios explained in the EpiRegio documentation: [evidence span withheld — anchor status: not_found]
