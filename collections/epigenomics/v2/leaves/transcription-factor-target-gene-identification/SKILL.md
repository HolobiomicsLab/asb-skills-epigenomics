---
name: transcription-factor-target-gene-identification
description: Use when you have a TF ChIP-seq peak file (e.g. an ENCODE BED file) and
  need the set of unique genes whose associated REMs overlap the TF's binding regions.
when_to_use_negative:
- The input is a list of genes of interest rather than genomic regions — use EpiRegio's
  Gene Query instead of the Region Query
- The goal is de novo identification of enriched TF motifs within a set of regions
  — use a motif enrichment tool such as PASTAA, Clover, MotifCounter, or Homer instead
- The input peaks are not in a coordinate format (BED) that the Region Query can compare
  against REMs by overlap percentage
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3656
tools:
- name: EpiRegio
  role: Web server hosting the Region Query that maps uploaded ChIP-seq peak BED regions
    to REMs with at least 50% overlap and returns associated genes; results exportable
    as CSV
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: ENCODE
  role: Source of the TF ChIP-seq peak BED file (e.g. ENCFF002CVL for ARID3A) used
    as Region Query input
- name: g:Profiler
  role: Optional downstream GO term enrichment analysis (default parameters) of the
    genes associated to the REMs recovered by the Region Query
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:6f23aa4dbd205d5f83ec9a5a04acfc9da4eb2c39b8ff9a5ea351a64218d3e42f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - ENCODE
  - EpiRegio
  - g:Profiler
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# transcription-factor-target-gene-identification

## Summary

Identifies the target genes of a transcription factor by querying ChIP-seq peak regions (BED) against EpiRegio's database of regulatory elements (REMs) using the Region Query, retrieving REM–gene associations filtered by an overlap-percentage criterion.

## When to use

Use when you have a TF ChIP-seq peak file (e.g. an ENCODE BED file) and need the set of unique genes whose associated REMs overlap the TF's binding regions.

## When NOT to use

- The input is a list of genes of interest rather than genomic regions — use EpiRegio's Gene Query instead of the Region Query
- The goal is de novo identification of enriched TF motifs within a set of regions — use a motif enrichment tool such as PASTAA, Clover, MotifCounter, or Homer instead
- The input peaks are not in a coordinate format (BED) that the Region Query can compare against REMs by overlap percentage

## Inputs

- TF ChIP-seq peak BED file (e.g. ENCODE accession ENCFF002CVL, 9026 peaks, decompressed)
- EpiRegio Region Query parameters (Overlap percentage = 50)

## Outputs

- REM–gene result table (REM identifiers, genomic coordinates, associated gene IDs), saved as CSV (e.g. arid3a_rem_gene_results.csv)
- Count of unique target genes (1721 for the ARID3A ENCFF002CVL example)

## How to apply

Download the TF's ChIP-seq binding locations from ENCODE (e.g. accession ENCFF002CVL for ARID3A, 9026 peaks) as a BED file and decompress it. Upload the unzipped BED file to EpiRegio's Region Query (https://epiregio.de/regionQuery/) and set the 'Overlap percentage' parameter to 50 so only REMs overlapping at least 50% of each binding location's length are returned. Export the resulting REM–gene table via the CSV download button and save it (e.g. as arid3a_rem_gene_results.csv). Count the distinct gene IDs in the exported table to obtain the number of unique target genes — the ARID3A example yielded REMs associated with 1721 unique genes. Optionally, rank REMs by their Cell type score and perform GO term enrichment of the resulting genes with g:Profiler to explore functionality.

## Related tools

- **EpiRegio** (Web server hosting the Region Query that maps uploaded ChIP-seq peak BED regions to REMs with at least 50% overlap and returns associated genes; results exportable as CSV) — https://github.com/TeamRegio/EpiRegioDB
- **ENCODE** (Source of the TF ChIP-seq peak BED file (e.g. ENCFF002CVL for ARID3A) used as Region Query input)
- **g:Profiler** (Optional downstream GO term enrichment analysis (default parameters) of the genes associated to the REMs recovered by the Region Query)

## Examples

```
curl -F file=@arid3a_ENCFF002CVL.bed -F overlap=50 https://epiregio.de/regionQuery/ > arid3a_rem_gene_results.csv
```

## Evaluation signals

- The exported CSV exists, is parseable as a tabular REM–gene result, and each REM row corresponds to a query hit passing the 50% overlap criterion
- The number of unique gene IDs in the result table matches the expected value (1721 unique genes for ARID3A ENCFF002CVL with 50% overlap)
- Every returned REM overlaps at least half the length of a query binding location
- The query peak count (9026 for ENCFF002CVL) matches the ENCODE accession's reported peak count
- The output table contains REM identifiers, genomic coordinates, and associated gene IDs as columns

## Limitations

- The linear model used in STITCHIT, which forms the basis for the Cell type score, is a simplification of the actual regulation of genes by their REMs
- Identifying REMs is inherently uncertain — no method yet locates them with absolute certainty, so EpiRegio REMs are putative regions explaining gene expression variation
- No tool versions are stated for the software mentioned (PASTAA, EpiRegio, etc.), so exact release reproducibility may be limited

## Evidence

- [methods] Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file: "Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file"
- [methods] Overlap percentage parameter: "Next to the upload field, you can see an option *Overlap percentage (optional)* to define the percentage that a REM should by minimum overlap"
- [methods] Region Query with 50% overlap: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [discussion] EpiRegio retrieves regulatory regions and their associated genes: "With EpiRegio we built an easy-to-access tool to efficiently retrieve regulatory regions and their associated genes."
- [abstract] EpiRegio is a resource of REMs: "Here we present the EpiRegio web server, a resource of regulatory elements (REMs)."
