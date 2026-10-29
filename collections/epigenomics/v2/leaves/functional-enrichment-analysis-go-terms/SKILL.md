---
name: functional-enrichment-analysis-go-terms
description: Use when you have a deduplicated list of gene IDs linked to a transcription
  factor's binding regions (e.g. from EpiRegio Region Query of ENCODE ChIP-seq peaks)
  and want to identify enriched GO terms or KEGG pathways explaining the TF's function.
when_to_use_negative:
- The input is already an enrichment result or pathway-term table rather than a raw
  gene ID list.
- The goal is to identify enriched TF binding motifs in DNA sequences rather than
  enriched functions of a gene list — use a motif enrichment tool such as PASTAA instead.
- The gene list contains duplicate IDs and cannot be deduplicated first, since duplicates
  are only considered once by g:Profiler and would misrepresent the input set.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3656
tools:
- name: g:Profiler
  role: Web tool that performs GO term and KEGG pathway enrichment analysis of the
    ARID3A-associated gene list with default parameters and Benjamini–Hochberg FDR
    correction at 0.05
  repo: https://biit.cs.ut.ee/gprofiler/gost
- name: EpiRegio
  role: Web server that links ChIP-seq peaks (ENCFF002CVL) to associated genes via
    its Region Query with a 50% overlap threshold, producing the gene list that serves
    as enrichment input
  repo: https://epiregio.de/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:e4f950ba211d058ebd16279b88a5a7fce27c5b7df5b54ff2687ed5e6f3af67f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - g:Profiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# functional-enrichment-analysis-go-terms

## Summary

Run GO term and KEGG pathway enrichment on a gene list derived from ChIP-seq-bound regulatory elements (e.g. ARID3A target genes from ENCODE peaks ENCFF002CVL) using g:Profiler with default parameters and Benjamini–Hochberg FDR 0.05, to infer biological function of the TF's regulatory network.

## When to use

Use when you have a deduplicated list of gene IDs linked to a transcription factor's binding regions (e.g. from EpiRegio Region Query of ENCODE ChIP-seq peaks) and want to identify enriched GO terms or KEGG pathways explaining the TF's function.

## When NOT to use

- The input is already an enrichment result or pathway-term table rather than a raw gene ID list.
- The goal is to identify enriched TF binding motifs in DNA sequences rather than enriched functions of a gene list — use a motif enrichment tool such as PASTAA instead.
- The gene list contains duplicate IDs and cannot be deduplicated first, since duplicates are only considered once by g:Profiler and would misrepresent the input set.

## Inputs

- Deduplicated list of gene IDs associated with ChIP-seq peaks (e.g. 1,721 ARID3A-associated genes from EpiRegio Region Query with 50% overlap of ENCFF002CVL)

## Outputs

- Enrichment table of significant GO terms and KEGG pathways with Benjamini–Hochberg adjusted p-values (exported as gprofiler_results.csv)
- Set of significant KEGG pathways and top biological processes at FDR <= 0.05

## How to apply

Obtain the gene list associated with the TF's ChIP-seq peaks (e.g. the 1,721 genes from the EpiRegio Region Query of ENCODE peaks ENCFF002CVL using a 50% overlap threshold), and deduplicate the gene IDs because g:Profiler considers duplicate gene IDs only once; if submitting via URL, note that only the first 90 unique genes fit in the URL, so paste the full list into the g:Profiler gost input field (https://biit.cs.ut.ee/gprofiler/gost). Run g:Profiler with default parameters on all gene IDs to perform GO term and KEGG pathway enrichment analysis. Apply a Benjamini–Hochberg adjusted p-value threshold of 0.05 to declare terms significant, then extract the significant pathways and top biological processes and export the enrichment table as gprofiler_results.csv.

## Related tools

- **g:Profiler** (Web tool that performs GO term and KEGG pathway enrichment analysis of the ARID3A-associated gene list with default parameters and Benjamini–Hochberg FDR correction at 0.05) — https://biit.cs.ut.ee/gprofiler/gost
- **EpiRegio** (Web server that links ChIP-seq peaks (ENCFF002CVL) to associated genes via its Region Query with a 50% overlap threshold, producing the gene list that serves as enrichment input) — https://epiregio.de/

## Examples

```
Navigate to https://biit.cs.ut.ee/gprofiler/gost, paste the 1,721 deduplicated ARID3A-associated gene IDs into the gost query field, run with default parameters, and export the results as gprofiler_results.csv
```

## Evaluation signals

- The significant terms at Benjamini–Hochberg FDR <= 0.05 must include the KEGG systemic lupus erythematosus pathway and the genes associated with alcoholism pathway.
- The top significant GO biological processes must include 'chromatin assembly or disassembly' and 'nucleosome organization', matching the enrichments reported in the paper.
- The gene input list must contain 1,721 unique gene IDs (deduplicated) before submission to g:Profiler.
- The exported gprofiler_results.csv contains the enrichment table with adjusted p-values at the 0.05 threshold.

## Limitations

- If submitting the gene list via a web URL, only the first 90 unique genes fit in the URL; the full list must be pasted into the g:Profiler gost input field instead.
- Duplicate gene IDs are only considered once by g:Profiler, so the input must be deduplicated to interpret results correctly.
- Results depend on the upstream Region Query parameters (e.g. the 50% REM–peak overlap threshold), so changing that threshold changes the gene set and hence the enrichment outcome.

## Evidence

- [results] Functional enrichment analysis (g:Profiler, default parameters, Benjamini–Hochberg FDR 0.05) of the 1,721 genes associated with ARID3A ChIP-seq peaks identified two enriched KEGG pathways: the systemic lupus erythematosus disease pathway and genes associated with alcoholism.: [evidence span withheld — anchor status: not_found]
- [methods] Run g:Profiler with default parameters on all 1,721 gene IDs to perform GO term and KEGG pathway enrichment analysis.: "perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs"
- [discussion] ARID3A is involved in regulating a network of genes found in pathways of SLE and alcoholism.: "ARID3A is involved in regulating a network of genes found in pathways of SLE and alcoholism"
- [readme] The README confirms the workflow application scenarios, including identifying TF target genes from ChIP-seq peak regions via EpiRegio documentation.: [evidence span withheld — anchor status: external_doc]
- [abstract] EpiRegio is a resource of regulatory elements (REMs), genomic regions associated with genes, from which the enrichment gene list is derived.: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
