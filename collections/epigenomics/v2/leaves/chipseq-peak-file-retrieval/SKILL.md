---
name: chipseq-peak-file-retrieval
description: Use when you have a transcription factor of interest and need its ChIP-seq
  peak regions as a BED file to query against a REM database such as EpiRegio for
  target-gene association.
when_to_use_negative:
- Input is a list of genes of interest rather than peak regions — use EpiRegio's Gene
  Query instead of the Region Query.
- The goal is motif/TF enrichment analysis of REM sequences (PASTAA, Clover, Homer)
  rather than mapping ChIP-seq peaks to REMs and target genes.
- No ENCODE (or equivalent) ChIP-seq peak file exists for the TF, so no BED binding
  locations can be supplied to the Region Query.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ENCODE
  role: Source database from which the TF ChIP-seq peak binding locations are downloaded
    as a BED file (accession ENCFF002CVL for ARID3A).
- name: EpiRegio
  role: Region Query web server that maps the uploaded BED peak file to REMs with
    at least 50% overlap and returns the REM–gene associations for export as CSV.
  repo: https://github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:ebd15a234a486132feea0939b0fe0c01917bfea2cf108b0bb71508f25760d5e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - ENCODE
  - EpiRegio
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chipseq-peak-file-retrieval

## Summary

Retrieve TF ChIP-seq binding locations as a BED file from ENCODE (e.g. ARID3A peaks under accession ENCFF002CVL), decompress them, and map them to EpiRegio REMs via the Region Query to identify candidate target genes.

## When to use

Use when you have a transcription factor of interest and need its ChIP-seq peak regions as a BED file to query against a REM database such as EpiRegio for target-gene association.

## When NOT to use

- Input is a list of genes of interest rather than peak regions — use EpiRegio's Gene Query instead of the Region Query.
- The goal is motif/TF enrichment analysis of REM sequences (PASTAA, Clover, Homer) rather than mapping ChIP-seq peaks to REMs and target genes.
- No ENCODE (or equivalent) ChIP-seq peak file exists for the TF, so no BED binding locations can be supplied to the Region Query.

## Inputs

- ENCODE accession number of a TF ChIP-seq peak file (e.g. ENCFF002CVL for ARID3A)
- Decompressed BED file of TF binding locations submitted to EpiRegio's Region Query

## Outputs

- BED file of ChIP-seq binding locations
- CSV result table of REMs overlapping the peaks (with REM identifiers, genomic coordinates, and associated gene IDs), e.g. arid3a_rem_gene_results.csv

## How to apply

Download the binding locations of the TF of interest from the ENCODE database as a BED file, using the accession number of the desired peak set (e.g. ENCFF002CVL for ARID3A ChIP-seq peaks, 9026 peaks). Decompress (unzip) the downloaded file so it is a plain BED file of genomic binding locations. Submit the unzipped BED file to EpiRegio's Region Query (https://epiregio.de/regionQuery/), setting the 'Overlap percentage' parameter to 50 so that only REMs overlapping at least 50% of each peak's length are returned. Export the resulting REM–gene result table as CSV (e.g. arid3a_rem_gene_results.csv) and count the unique associated gene IDs as the downstream mapping output.

## Related tools

- **ENCODE** (Source database from which the TF ChIP-seq peak binding locations are downloaded as a BED file (accession ENCFF002CVL for ARID3A).)
- **EpiRegio** (Region Query web server that maps the uploaded BED peak file to REMs with at least 50% overlap and returns the REM–gene associations for export as CSV.) — https://github.com/TeamRegio/EpiRegioDB

## Evaluation signals

- The downloaded ENCODE file decompresses successfully into a parseable BED file of genomic binding locations (e.g. 9026 peaks for ENCFF002CVL).
- The Region Query output file (e.g. arid3a_rem_gene_results.csv) exists and is parseable as a tabular REM–gene result.
- Every returned REM row corresponds to a query hit satisfying the applied 50% overlap criterion against a submitted peak.
- The number of unique gene IDs in the result table is consistent with the peak set (for ENCFF002CVL: 1721 unique genes reported).
- The overlap percentage parameter was explicitly set (50) rather than left at the default, and the CSV export was produced via the table's CSV download.

## Limitations

- REMs are only returned when they overlap at least 50% of a peak's length, so peaks with weaker REM coverage are dropped from the mapping.
- The EpiRegio REM set covers specific human primary cell types and tissues; REM–gene associations depend on chromatin accessibility variation linked to gene expression and may not generalize across cell types.
- The linear model used in STITCHIT, which underlies the Cell type score, is a simplification of actual gene regulation by REMs.

## Evidence

- [methods] Download binding locations as BED from ENCODE: "Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file"
- [methods] Region Query overlap criterion: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [methods] ARID3A example accession: "we use the ChIP-seq peaks of TF ARID3A with the accession number ENCFF002CVL"
- [readme] Application scenario for TF target genes: [evidence span withheld — anchor status: external_doc]
