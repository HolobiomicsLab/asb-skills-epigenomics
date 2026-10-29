---
name: geo-chipseq-raw-data-retrieval
description: Use when a downstream module requires raw ChIP-seq reads rather than
  processed peaks — e.g. deriving H3K27me3 domain-breadth cutoffs per cell type from
  GEO/SRA accessions with matched input controls.
when_to_use_negative:
- When processed peak calls or signal tracks are already available and only gene-level
  overlap is needed — raw FASTQ retrieval is unnecessary.
- When the analysis does not require a matched input control (e.g. reusing existing
  MACS2 broad peak BED files), fetching the paired input runs adds no value.
- When working outside the ESC/NCC/cardiomyocyte context or non-hg19 assemblies without
  re-deriving the per-dataset findiplist() cutoffs.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: GEO
  role: Public repository from which the H3K27me3 ChIP-seq and matched input control
    FASTQs are downloaded
- name: MACS2
  role: 'Downstream consumer of the retrieved FASTQ pairs: calls broad H3K27me3 peaks
    in broad mode using the matched input control on hg19'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:d60fb16b7ba978fa51a4cda3f1148ff549630af1dbf932663fa365005be4c447
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  tools:
  - GEO
  - MACS2
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# geo-chipseq-raw-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve raw H3K27me3 ChIP-seq FASTQ files (and their matched input controls) from GEO/SRA for specific ESC, neural crest cell and cardiomyocyte datasets, as required to reconstruct POSTRE's polycombScore module with hg19-based broad peak calling.

## When to use

Use when a downstream module requires raw ChIP-seq reads rather than processed peaks — e.g. deriving H3K27me3 domain-breadth cutoffs per cell type from GEO/SRA accessions with matched input controls.

## When NOT to use

- When processed peak calls or signal tracks are already available and only gene-level overlap is needed — raw FASTQ retrieval is unnecessary.
- When the analysis does not require a matched input control (e.g. reusing existing MACS2 broad peak BED files), fetching the paired input runs adds no value.
- When working outside the ESC/NCC/cardiomyocyte context or non-hg19 assemblies without re-deriving the per-dataset findiplist() cutoffs.

## Inputs

- GEO accession identifiers: GSE24447, GSE108518, GSE85628
- SRA run accessions: SRR067372/SRR067371 (ESC), SRR6418921/SRR6418919 (NCC), SRR4032228/SRR4032231 (cardiomyocytes)
- hg19 reference genome coordinates used by all downstream POSTRE steps

## Outputs

- H3K27me3 ChIP-seq FASTQ files per cell type (ESC, NCC, cardiomyocytes)
- Matched input control FASTQ files per dataset
- Paired ChIP/control file set ready for MACS2 broad peak calling

## How to apply

Identify the GEO accession and its paired ChIP-seq/input-control SRR accessions per cell type: GSE24447 (SRR067372 H3K27me3 / SRR067371 input) for ESC, GSE108518 (SRR6418921 / SRR6418919) for neural crest cells, and GSE85628 (SRR4032228 / SRR4032231) for cardiomyocytes. Download the H3K27me3 ChIP-seq and matched input control FASTQs from GEO/SRA for each pair, keeping ChIP and control strictly matched per dataset. Align and process on hg19 coordinates, since POSTRE's downstream peak merging, cutoff derivation with findiplist(), and gene-body overlap are all defined in hg19. The rationale for fetching raw data (not peak files) is that the peak-size cutoff (6689 bp ESC, 6719 bp NCC, 8020 bp cardiomyocytes) is derived de novo from merged broad peaks called by MACS2 in broad mode, which requires the original reads and matched controls.

## Related tools

- **GEO** (Public repository from which the H3K27me3 ChIP-seq and matched input control FASTQs are downloaded)
- **MACS2** (Downstream consumer of the retrieved FASTQ pairs: calls broad H3K27me3 peaks in broad mode using the matched input control on hg19)

## Evaluation signals

- All six SRR runs are retrieved and correctly paired as ChIP/input: SRR067372+SRR067371 (GSE24447), SRR6418921+SRR6418919 (GSE108518), SRR4032228+SRR4032231 (GSE85628).
- Each ChIP FASTQ set has a matched input control from the same dataset, required for MACS2 broad-mode calling.
- Retrieved data maps cleanly to hg19, matching the coordinate system used for bedtools merge and hg19 gene-body overlap.
- Downstream MACS2 broad peak calling succeeds and merged-peak length distributions yield findiplist() cutoffs near 6689 bp (GSE24447), 6719 bp (GSE108518) and 8020 bp (GSE85628).

## Limitations

- POSTRE's genomic inputs are bulk datasets from whole tissues/structures; e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so analogous granularity limits apply to the H3K27me3 datasets retrieved.
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect the relevance of the retrieved cell-type data for a given disease context.

## Evidence

- [supplementary] Download the H3K27me3 ChIP-seq and matched input control FASTQs from GEO/SRA for GSE24447 (SRR067372/SRR067371), GSE108518: [evidence span withheld — anchor status: not_found]
- [supplementary] Call broad H3K27me3 peaks with MACS2 in broad mode using matched input control and hg19 coordinates: [evidence span withheld — anchor status: not_found]
- [supplementary] cutoffs fall near 6689/6719/8020 bp: [evidence span withheld — anchor status: not_found]
