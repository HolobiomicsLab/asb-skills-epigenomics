---
name: regulatory-element-gene-association-query
description: Use when you have TF ChIP-seq peak BED files or a list of differentially
  expressed genes and need the REMs associated with them, with optional cell-type/tissue
  filtering.
when_to_use_negative:
- You need REMs identified de novo from raw chromatin accessibility data rather than
  retrieval of pre-computed REM–gene associations from EpiRegio's database
- Your regions are not in a supported genomic coordinate format (BED) or your gene
  identifiers cannot be matched by EpiRegio's Gene Query, which causes substantial
  shortfalls in returned REMs
- You want TF motif discovery directly — that requires a follow-up PASTAA run on extracted
  REM sequences, not the query itself
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: EpiRegio
  role: Web server (Region Query and Gene Query, plus REST API) that retrieves REMs
    associated with ChIP-seq peak regions or genes of interest, with Cell type score
    ranking and cell type/tissue filtering
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: ENCODE
  role: Source database for downloading TF binding locations as BED files (e.g. ARID3A
    accession ENCFF002CVL) used as Region Query input
- name: g:Profiler
  role: Downstream GO term / KEGG pathway enrichment (default parameters, Benjamini–Hochberg
    FDR 0.05) of the unique genes retrieved by the REM–gene query
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:8e6dc30793703252bb968c8117f13d3da7e9857379de80b4bc5819c73afa54e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - ENCODE
  - EpiRegio
  - Seurat
  - g:Profiler
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# regulatory-element-gene-association-query

## Summary

Query EpiRegio to retrieve regulatory elements (REMs) linked to genes of interest, either by submitting ChIP-seq peak BED files (Region Query) or differential-expression gene lists (Gene Query), returning REM–gene association tables with genomic coordinates and cell-type-specific scores.

## When to use

Use when you have TF ChIP-seq peak BED files or a list of differentially expressed genes and need the REMs associated with them, with optional cell-type/tissue filtering.

## When NOT to use

- You need REMs identified de novo from raw chromatin accessibility data rather than retrieval of pre-computed REM–gene associations from EpiRegio's database
- Your regions are not in a supported genomic coordinate format (BED) or your gene identifiers cannot be matched by EpiRegio's Gene Query, which causes substantial shortfalls in returned REMs
- You want TF motif discovery directly — that requires a follow-up PASTAA run on extracted REM sequences, not the query itself

## Inputs

- BED file of TF ChIP-seq peaks (e.g. ENCODE accession ENCFF002CVL), unzipped
- Text file of differentially expressed gene IDs (e.g. GeneSet.txt, matching the example in ApplicationScenarioExamples/identifyEnrichedTFs)
- Overlap percentage parameter (e.g. 50) for Region Query
- Cell type/tissue filter string (e.g. 'heart') for Gene Query

## Outputs

- CSV table of REM identifiers, genomic coordinates, and associated gene IDs (e.g. arid3a_rem_gene_results.csv)
- Gene Query CSV of REMs associated with input genes, filtered by cell type/tissue
- Unique gene count from REM–gene mappings (e.g. 1,721 genes for ARID3A peaks)
- REM sequences for downstream motif analysis (via bedtools getfasta)

## How to apply

For a Region Query, download TF binding locations (e.g. ARID3A ChIP-seq peaks ENCFF002CVL from ENCODE) as an unzipped BED file, submit to https://epiregio.de/regionQuery/ and set the 'Overlap percentage' parameter (50 means a REM must cover at least half of each binding location's length); export the REM–gene result table as CSV. For a Gene Query, submit a gene list (e.g. 304 TGF-β-treated versus untreated HUVEC DE genes from Seurat's FindAllMarkers with Wilcoxon Rank Sum, P ≤ 0.01) to https://epiregio.de/geneQuery/ or the REST API https://epiregio.de/REST_API/GeneQuery/<input>/, optionally entering a tissue such as 'heart' in the 'Filter for cell types/tissues' field, then download the table via the CSV button. Deduplicate gene IDs before downstream enrichment (g:Profiler counts duplicates once), and optionally rank returned REMs by their Cell type score. Results can feed GO/KEGG enrichment (g:Profiler, Benjamini–Hochberg FDR 0.05) or PASTAA motif enrichment on REM sequences extracted with bedtools getfasta.

## Related tools

- **EpiRegio** (Web server (Region Query and Gene Query, plus REST API) that retrieves REMs associated with ChIP-seq peak regions or genes of interest, with Cell type score ranking and cell type/tissue filtering) — https://github.com/TeamRegio/EpiRegioDB
- **ENCODE** (Source database for downloading TF binding locations as BED files (e.g. ARID3A accession ENCFF002CVL) used as Region Query input)
- **g:Profiler** (Downstream GO term / KEGG pathway enrichment (default parameters, Benjamini–Hochberg FDR 0.05) of the unique genes retrieved by the REM–gene query)

## Examples

```
curl -X POST https://epiregio.de/REST_API/GeneQuery/GeneSet.txt/  # or upload arid3a_peaks.bed at https://epiregio.de/regionQuery/ with Overlap percentage = 50
```

## Evaluation signals

- Region Query with 50% overlap on ARID3A peaks (ENCFF002CVL, 9,026 peaks) must yield REMs associated with 1,721 unique genes
- Gene Query on 304 TGF-β-treated vs untreated HUVEC DE genes (Seurat FindAllMarkers, Wilcoxon Rank Sum, P ≤ 0.01) filtered for 'heart' must return 11,836 REMs (metric_rems_de_genes under cond_tgfb_huvec_endomt)
- Exported CSV must exist, be parseable as a tabular REM–gene result, and its REM rows must correspond to query hits with the applied 50% overlap criterion
- Downstream g:Profiler results at Benjamini–Hochberg FDR ≤ 0.05 must include KEGG SLE and alcoholism pathways and GO biological processes chromatin assembly or disassembly and nucleosome organization
- A substantial shortfall in returned REMs indicates incorrect gene identifiers or filtering settings

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs
- When submitting via a web URL, only the first 90 unique genes fit in the URL — the full list must be pasted into the g:Profiler gost input field; duplicates are only considered once by g:Profiler
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.)

## Evidence

- [methods] Region Query with 50% overlap: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [methods] gene query cell type filter: "Enter *heart* in the field *Filter for cell types/tissues*"
- [methods] overlap percentage option: "Next to the upload field, you can see an option *Overlap percentage (optional)* to define the percentage that a REM should by minimum overlap"
- [abstract] EpiRegio REM resource: "Here we present the EpiRegio web server, a resource of regulatory elements (REMs)."
- [readme] application scenario examples repo: [evidence span withheld — anchor status: external_doc]
