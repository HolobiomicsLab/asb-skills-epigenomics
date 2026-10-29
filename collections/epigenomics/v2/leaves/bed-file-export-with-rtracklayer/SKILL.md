---
name: bed-file-export-with-rtracklayer
description: Use when a GRanges object of differential regions (e.g., a DiffBind dba.report
  result) must be written out as a BED file to feed region-set tools like BIT.
when_to_use_negative:
- The query regions are already in BED/peak format (e.g., MACS2 narrowPeak output)
  — BIT can consume them directly without re-export.
- The analysis is single-cell or RNA-seq based and produces no genomic coordinate
  ranges to export.
- The genome assembly is not mm10/hg38-mappable — coordinates must match the TR ChIP-seq
  reference library.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: rtracklayer
  role: Exports the DAR GRanges object to BED format (export.bed) and validates BED
    importability
- name: DiffBind
  role: Produces the dba.report differential table whose regions are converted to
    GRanges and exported
- name: BIT
  role: Downstream consumer of the exported BED region set as its epigenomic query
    input
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:458cb53b55a36595763022be78635f995a6c2bf4ff1febe52ab4d26147e53768
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BIT
  - DESeq2
  - DiffBind
  - R
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# BED file export with rtracklayer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports differentially accessible region (DAR) GRanges objects — e.g., the RUNX1 cKO-vs-WT DAR set produced by DiffBind — to BED format (mm10_RUNX1.bed) so they can serve as epigenomic query region sets for BIT.

## When to use

Use when a GRanges object of differential regions (e.g., a DiffBind dba.report result) must be written out as a BED file to feed region-set tools like BIT.

## When NOT to use

- The query regions are already in BED/peak format (e.g., MACS2 narrowPeak output) — BIT can consume them directly without re-export.
- The analysis is single-cell or RNA-seq based and produces no genomic coordinate ranges to export.
- The genome assembly is not mm10/hg38-mappable — coordinates must match the TR ChIP-seq reference library.

## Inputs

- GRanges object (or dba.report data frame with Fold, p.value, FDR columns) of differential regions from DiffBind
- mm10 genomic coordinates from ATAC-seq consensus peak analysis

## Outputs

- mm10_RUNX1.bed — BED file of DARs importable by rtracklayer (~15,652 ranges spanning mm10 chromosomes)
- Optionally separate gained-accessibility (Fold > 0) and lost-accessibility (Fold < 0) BED files

## How to apply

After running the DiffBind pipeline (dba.count with minOverlap = 2, dba.blacklist with DBA_BLACKLIST_MM10, DESeq2 normalization and analysis), obtain the differential report with dba.report at th = 0.1 and convert the resulting rows into GRanges. Then use rtracklayer's export.bed to write the DAR set as a BED file (e.g., mm10_RUNX1.bed), splitting regions by Fold > 0 (gained accessibility) and Fold < 0 (lost accessibility) if gained/lost sets are needed separately. The rationale is that BIT only requires a set of epigenomic regions containing chromosome numbers and coordinates as input, so the exported BED must faithfully preserve mm10 coordinates from the differential analysis.

## Related tools

- **rtracklayer** (Exports the DAR GRanges object to BED format (export.bed) and validates BED importability)
- **DiffBind** (Produces the dba.report differential table whose regions are converted to GRanges and exported)
- **BIT** (Downstream consumer of the exported BED region set as its epigenomic query input) — https://github.com/ZeyuL01/BIT

## Examples

```
rtracklayer::export.bed(dar_granges, "mm10_RUNX1.bed")
```

## Evaluation signals

- The exported BED contains the expected number of GRanges (e.g., 15,652 ranges spanning mm10 chromosomes).
- The BED file is re-importable by rtracklayer (import.bed round-trip preserves chromosome, start, end).
- Regions in the BED match the dba.report table rows, with Fold, p.value, and FDR columns available.
- Gained (Fold > 0) and lost (Fold < 0) subsets partition the full DAR set without overlap or loss of regions.
- The exported file carries no ENCODE blacklist regions, since dba.blacklist with DBA_BLACKLIST_MM10 was applied upstream.

## Limitations

- BIT's performance depends on NGS data quality; sequencing biases such as GC content bias and PCR amplification bias can reduce the reliability of the input epigenomic region set.
- Exported coordinates are assembly-specific (mm10 here); using them against a differently-built TR binding reference library will misalign regions.
- BIT can suffer from biological complexities of TR regulation, which can reduce the method's performance regardless of correct BED export.

## Evidence

- [methods] export the DAR set with rtracklayer's export.bed as mm10_RUNX1.bed: [evidence span withheld — anchor status: not_found]
- [readme] user-provided set of epigenomic regions: [evidence span withheld — anchor status: external_doc]
