---
name: gff-transcript-partitioning
description: Use when you need fixed genomic windows for CLIP-seq binding-site testing
  over annotated transcripts (e.g., GENCODE GFF3 for Skipper) instead of peak calling,
  ensuring no window spans intron/exon or gene boundaries.
when_to_use_negative:
- The GFF has not been filtered/processed to select one best transcript per gene —
  raw multi-isoform GENCODE GFFs make window selection suboptimal and results uninterpretable
- You need peak calling on unannotated or poorly annotated transcriptomes where fixed
  windows over annotated transcripts would miss binding sites
- The analysis requires variable-width peaks or windows spanning exon-intron boundaries
  rather than Skipper's annotation-aware fixed tiling
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline whose GFF-partitioning stage generates the tiled
    windows (PARTITION output) from the filtered GFF
  repo: https://github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine that orchestrates the GFF partitioning step and auto-generates
    the partition BED during the Skipper run
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:a3f543e7bf6b7ea7ffb10180adee33af751408ff6bc3306d16a7bc7daa9b68be
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper
  tools:
  - SLURM
  - Skipper
  - Snakemake 9.12.0
  - conda
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gff-transcript-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions an annotated transcriptome (filtered GENCODE GFF3) into disjoint windows of at most 100 nt that never cross exon-intron junctions or gene boundaries, replacing peak calling in the Skipper eCLIP workflow with fixed candidate binding-site windows.

## When to use

Use when you need fixed genomic windows for CLIP-seq binding-site testing over annotated transcripts (e.g., GENCODE GFF3 for Skipper) instead of peak calling, ensuring no window spans intron/exon or gene boundaries.

## When NOT to use

- The GFF has not been filtered/processed to select one best transcript per gene — raw multi-isoform GENCODE GFFs make window selection suboptimal and results uninterpretable
- You need peak calling on unannotated or poorly annotated transcriptomes where fixed windows over annotated transcripts would miss binding sites
- The analysis requires variable-width peaks or windows spanning exon-intron boundaries rather than Skipper's annotation-aware fixed tiling

## Inputs

- Filtered GENCODE or Ensembl GFF3 annotation (e.g., gencode.v49.basic.annotation.gff3.gz) with one best transcript per gene
- ACCESSION_RANKINGS file ranking gene and transcript feature types for resolving overlapping annotations

## Outputs

- Gzipped, sorted, tab-delimited BED file of disjoint, uniquely named windows (each <=100 nt) that never cross intron/exon or gene boundaries (the Skipper PARTITION output)

## How to apply

Load the filtered GENCODE GFF (e.g., gencode.v49.basic.annotation.gff3.gz produced by gff_transcript_quality_filter.R in gencode mode) and an ACCESSION_RANKINGS file ranking gene/transcript feature types. Iterate over transcript-quality-ranked features (GENCODE hierarchy such as MANE_Select, Ensembl_canonical, GENCODE_Primary), selecting one best transcript per gene so window selection is optimal and interpretable. Tile consecutive windows of at most 100 nt along each selected transcript, splitting at exon-intron junctions and gene boundaries so no window spans a boundary. Resolve regions assigned to multiple feature types using the accession type ranking file, assign unique window IDs, and write disjoint, sorted, tab-delimited BED output (gzipped) as the PARTITION file. This fixed-window design is why Skipper 'skips peak calling by tiling windows over annotated transcripts'.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline whose GFF-partitioning stage generates the tiled windows (PARTITION output) from the filtered GFF) — https://github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine that orchestrates the GFF partitioning step and auto-generates the partition BED during the Skipper run)

## Examples

```
snakemake -s Skipper.py --configfile Example_config.yaml --profile profiles/example_slurm  # auto-generates the GFF partition (tiled <=100-nt windows) from gencode.v49.basic.annotation.gff3.gz
```

## Evaluation signals

- Output is a valid gzipped BED file in sorted order
- All windows are pairwise disjoint (non-overlapping)
- Every window is at most 100 nt long
- Each window has a unique identifier
- No window crosses an intron/exon junction or gene boundary as defined in the input GFF

## Limitations

- Skipper attempts to avoid windows crossing intron/exon boundaries, so binding signal straddling boundaries may be split across windows
- Requires a properly filtered GFF: 'it is incredibly important to ensure that these GFF files have been properly cleaned and filtered before supplying them to Skipper'
- The first Skipper run that parses the GFF and builds the partition plus STAR index is slow; subsequent runs are faster

## Evidence
