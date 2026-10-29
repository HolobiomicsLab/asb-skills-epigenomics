---
name: genomic-region-overlap-querying
description: Use when you have TF binding locations (e.g. an ENCODE ChIP-seq peak
  BED file) and need to find which EpiRegio REMs overlap them and which target genes
  those REMs are associated with.
when_to_use_negative:
- You already have REM identifiers or genes of interest and should use EpiRegio's
  Gene Query instead of mapping regions
- You need REMs covering less than half of each peak's length (a lower overlap threshold
  is required than the 50% default used here)
- You want de novo identification of regulatory elements rather than mapping known
  binding regions onto EpiRegio's REM collection
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: EpiRegio
  role: Web server hosting the Region Query that maps uploaded BED binding locations
    onto REMs with a user-defined minimum overlap percentage and links them to target
    genes
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: ENCODE
  role: Source database for downloading the TF ChIP-seq peak BED file (e.g. ARID3A
    accession ENCFF002CVL)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:754053839bc712b6ba20086a93824a81c677b073acbe19199b163d8b35bf49e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - ENCODE
  - EpiRegio
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-region-overlap-querying

## Summary

Maps TF ChIP-seq peak regions (BED) onto EpiRegio's regulatory elements (REMs) using the Region Query with a minimum overlap percentage, to retrieve REM–gene associations for downstream target-gene analyses.

## When to use

Use when you have TF binding locations (e.g. an ENCODE ChIP-seq peak BED file) and need to find which EpiRegio REMs overlap them and which target genes those REMs are associated with.

## When NOT to use

- You already have REM identifiers or genes of interest and should use EpiRegio's Gene Query instead of mapping regions
- You need REMs covering less than half of each peak's length (a lower overlap threshold is required than the 50% default used here)
- You want de novo identification of regulatory elements rather than mapping known binding regions onto EpiRegio's REM collection

## Inputs

- Unzipped BED file of TF ChIP-seq peak binding locations (e.g. ENCODE accession ENCFF002CVL)
- Overlap percentage parameter (50 in the ARID3A application scenario)

## Outputs

- CSV result table of REM identifiers, genomic coordinates, and associated gene IDs (e.g. arid3a_rem_gene_results.csv)
- Set of unique target genes associated with overlapping REMs (1721 unique genes for the ARID3A ENCFF002CVL scenario)

## How to apply

Download the TF binding locations of interest from the ENCODE database as a BED file (e.g. ARID3A ChIP-seq peaks, accession ENCFF002CVL, 9026 peaks) and unzip it. Submit the BED file to EpiRegio's Region Query (https://epiregio.de/regionQuery/) and set the 'Overlap percentage (optional)' parameter to 50 so only REMs overlapping at least 50% of each binding location's length are returned. Export the resulting table by clicking the bottom CSV button, yielding REM identifiers, genomic coordinates, and associated gene IDs. This mapping is a prerequisite for downstream steps such as extracting REM sequences with bedtools getfasta for PASTAA motif enrichment, or GO term enrichment with g:Profiler on the resulting genes.

## Related tools

- **EpiRegio** (Web server hosting the Region Query that maps uploaded BED binding locations onto REMs with a user-defined minimum overlap percentage and links them to target genes) — https://github.com/TeamRegio/EpiRegioDB
- **ENCODE** (Source database for downloading the TF ChIP-seq peak BED file (e.g. ARID3A accession ENCFF002CVL))

## Examples

```
Submit the unzipped ENCFF002CVL BED file at https://epiregio.de/regionQuery/ with 'Overlap percentage' = 50, then export the result via the CSV button as arid3a_rem_gene_results.csv
```

## Evaluation signals

- Output CSV exists and is parseable as a tabular REM–gene result
- Returned REM rows correspond to query hits satisfying the applied 50% overlap criterion
- Result table contains REM identifiers, genomic coordinates, and associated gene IDs
- Reproducing the ARID3A ENCFF002CVL scenario (9026 peaks, 50% overlap) yields REMs associated with 1721 unique genes

## Limitations

- The linear model used in STITCHIT, which underlies EpiRegio's Cell type score, is a simplification of actual regulation of genes by their REMs
- Results depend on the chosen overlap percentage; using 50% restricts hits to REMs covering at least half of each binding location's length
- No tool versions are stated for the software in the workflow

## Evidence

- [methods] Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file: "Download the binding locations of the TF of interest, for instance from the ENCODE database as a BED file"
- [methods] Overlap percentage option: "Next to the upload field, you can see an option *Overlap percentage (optional)* to define the percentage that a REM should by minimum overlap"
- [methods] Region query with 50% overlap: "search for REMs overlapping at least 50% with the TF ChIP-seq peaks"
- [methods] CSV export of results: "Download the resulting table by clicking on the bottom *CSV*."
- [methods] ARID3A ChIP-seq accession: "we use the ChIP-seq peaks of TF ARID3A with the accession number ENCFF002CVL"
- [discussion] ARID3A target gene network finding: [evidence span withheld — anchor status: not_found]
- [readme] EpiRegioDB repository reference: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
