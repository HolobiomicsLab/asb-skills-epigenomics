---
name: bed-coordinate-manipulation
description: Use when running Skipper's GFF partitioning step on a GENCODE/Ensembl
  GFF3 (e.g. gencode.v49.basic.annotation.gff3.gz) to produce the fixed <=100-nt window
  BED used for CLIP-seq binding site detection.
when_to_use_negative:
- The analysis goal is de novo peak discovery without a genome/transcript annotation
  — Skipper's tiling depends on an annotated GFF and skips peak calling by design
- Windows must span exon-intron or gene boundaries (e.g. genomic-region-level analyses),
  since Skipper windows are explicitly split at these boundaries
- The GFF is raw and unfiltered (many alternative isoforms per gene); filter to one
  best transcript per gene first or window selection will not be interpretable
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Skipper
  role: 'Performs the GFF partitioning: tiles annotated transcripts into <=100-nt
    windows avoiding intron/exon and gene boundaries, emitting the PARTITION BED'
  repo: github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow manager that executes the Skipper pipeline including the GFF partitioning
    rule
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:dcfa76c0285e9a3109fcb35964fd79b42086f1d30a1d59d8a0832a58e965adc2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - Skipper
  - Snakemake 9.12.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bed-coordinate-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates Skipper's transcriptome partition by tiling the filtered GENCODE GFF into disjoint windows of at most 100 nt that never cross exon-intron junctions or gene boundaries, replacing peak calling with a fixed window framework for eCLIP binding site detection.

## When to use

Use when running Skipper's GFF partitioning step on a GENCODE/Ensembl GFF3 (e.g. gencode.v49.basic.annotation.gff3.gz) to produce the fixed <=100-nt window BED used for CLIP-seq binding site detection.

## When NOT to use

- The analysis goal is de novo peak discovery without a genome/transcript annotation — Skipper's tiling depends on an annotated GFF and skips peak calling by design
- Windows must span exon-intron or gene boundaries (e.g. genomic-region-level analyses), since Skipper windows are explicitly split at these boundaries
- The GFF is raw and unfiltered (many alternative isoforms per gene); filter to one best transcript per gene first or window selection will not be interpretable

## Inputs

- Filtered GENCODE or Ensembl GFF3 annotation (e.g. gencode.v49.basic.annotation.gff3.gz), one best transcript per gene
- ACCESSION_RANKINGS file ranking gene and transcript feature types (e.g. MANE_Select, Ensembl_canonical, GENCODE_Primary)
- Reference genome FASTA (e.g. GRCh38.primary_assembly.genome.fa) used in the Skipper pipeline

## Outputs

- Gzipped, sorted, tab-delimited BED file of disjoint windows, each <=100 nt, uniquely named (the PARTITION output)
- Windows annotated with a single primary feature type (exon, intron, UTR, etc.)

## How to apply

Load the filtered GENCODE GFF and the ACCESSION_RANKINGS file, then iterate over transcript-quality-ranked features (GENCODE hierarchy such as MANE_Select, Ensembl_canonical, GENCODE_Primary), selecting one best transcript per gene using the GFF 'tag' column so window selection stays optimal and interpretable. Tile consecutive windows of at most 100 nt along each selected transcript, splitting at exon-intron junctions and gene boundaries so no window spans a boundary (Skipper deliberately avoids creating windows that cross intron/exon boundaries). Resolve regions still assigned to multiple feature types using the accession type ranking to give each window a single primary feature type. Assign each window a unique identifier and write the disjoint, sorted, tab-delimited, gzipped BED as the PARTITION output. This replaces peak calling because Skipper analyzes read start signal in these fixed windows using a discrete annotation framework.

## Related tools

- **Skipper** (Performs the GFF partitioning: tiles annotated transcripts into <=100-nt windows avoiding intron/exon and gene boundaries, emitting the PARTITION BED) — github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow manager that executes the Skipper pipeline including the GFF partitioning rule)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file is valid gzipped BED and parses as sorted, tab-delimited records
- All windows are disjoint (no overlaps) and each window length is <=100 nt
- Every window has a unique identifier/name
- No window crosses an exon-intron junction or gene boundary present in the input GFF
- Each window resolves to exactly one primary feature type per the accession ranking file

## Limitations

- Windows never traverse exon-intron junctions or gene boundaries, so signal spanning such boundaries is split across windows rather than evaluated jointly
- Requires a properly filtered GFF with one 'best' transcript per gene (selected via the tag column, e.g. MANE_Select/Ensembl_canonical/GENCODE_Primary); raw multi-isoform GFFs yield suboptimal, hard-to-interpret windows
- Downstream Skipper statistical testing additionally requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking such controls cannot be analyzed

## Evidence

- [intro] Here, we introduce an end-to-end solution for analyzing CLIP-seq data (Skipper) that skips peak calling by tiling windows over annotated transcripts.: "Here, we introduce an end-to-end solution for analyzing CLIP-seq data (Skipper) that skips peak calling by tiling windows over annotated transcripts."
- [readme] Skipper avoids windows crossing intron/exon boundaries: "Skipper uses GFF files both to provide users with metadata on the types of genomic features an RBP is binding (e.g., introns, exons, UTRs, etc.)"
- [methods] Best transcript per gene via tag column: "transcripts are filtered using the "tag" column of the GFF file, where we attempt to select the best transcript for each gene"
- [methods] Overlapping feature annotation resolution: "Second, Skipper resolves overlapping feature annotations."
