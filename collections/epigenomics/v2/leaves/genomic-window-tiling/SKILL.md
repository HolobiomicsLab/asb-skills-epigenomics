---
name: genomic-window-tiling
description: Use when generating fixed candidate binding-site windows over GENCODE/Ensembl
  GFF annotations for CLIP-seq analysis (Skipper-style), replacing peak calling with
  annotation-aware tiling.
when_to_use_negative:
- You need de novo peak calling on unannotated or poorly annotated transcriptomes
  where GFF transcript selection per gene is unreliable.
- You only have raw CLIP reads without a matched input sample or a properly filtered
  GFF — Skipper requires a matched input control to build its beta-binomial model
  downstream.
- Windows must span exon-intron junctions or gene boundaries by design (e.g. whole-gene
  or whole-transcript aggregations).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline whose GFF partitioning step performs the annotation-aware
    <=100-nt window tiling
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine (v9.12.0) that orchestrates the GFF partitioning rule producing
    the PARTITION BED output
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:95ce72d2d6be89084c137d6564b41acb8e7852a4db1df5c729d849e5b70e3562
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
  - Skipper
  - Snakemake
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-window-tiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tile annotated transcripts into disjoint windows of at most 100 nt (Skipper's GFF partitioning), so CLIP-seq binding can be tested in fixed windows instead of peak calling, with no window crossing exon-intron junctions or gene boundaries.

## When to use

Use when generating fixed candidate binding-site windows over GENCODE/Ensembl GFF annotations for CLIP-seq analysis (Skipper-style), replacing peak calling with annotation-aware tiling.

## When NOT to use

- You need de novo peak calling on unannotated or poorly annotated transcriptomes where GFF transcript selection per gene is unreliable.
- You only have raw CLIP reads without a matched input sample or a properly filtered GFF — Skipper requires a matched input control to build its beta-binomial model downstream.
- Windows must span exon-intron junctions or gene boundaries by design (e.g. whole-gene or whole-transcript aggregations).

## Inputs

- Filtered GENCODE GFF3 annotation (e.g. gencode.v49.basic.annotation.gff3.gz)
- ACCESSION_RANKINGS file ranking gene/transcript accession types (GENCODE tag hierarchy: MANE_Select, Ensembl_canonical, GENCODE_Primary)

## Outputs

- Gzipped, sorted, tab-delimited BED file of disjoint, uniquely named windows, each <=100 nt, none spanning an intron/exon or gene boundary (Skipper PARTITION output)

## How to apply

Load the filtered GENCODE GFF (e.g. gencode.v49.basic.annotation.gff3.gz) and an accession/rankings file that orders gene and transcript types. Iterate over transcript-quality-ranked features (GENCODE hierarchy such as MANE_Select, Ensembl_canonical, GENCODE_Primary) to select one best transcript per gene, keeping window selection optimal and interpretable. Tile consecutive windows of at most 100 nt along each selected transcript, splitting at exon-intron junctions and gene boundaries so no window spans a boundary. Resolve regions still assigned to multiple feature types using the accession type ranking to assign one primary feature type per window, then assign unique identifiers and write the disjoint windows as a sorted, gzipped, tab-delimited BED PARTITION output.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline whose GFF partitioning step performs the annotation-aware <=100-nt window tiling) — https://github.com/YeoLab/skipper
- **Snakemake** (Workflow engine (v9.12.0) that orchestrates the GFF partitioning rule producing the PARTITION BED output) — https://github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output is a valid gzipped BED file in sorted, tab-delimited format.
- All windows are disjoint and each is <=100 nt in length.
- Every window has a unique identifier.
- No window crosses an exon-intron junction or gene boundary as defined in the input GFF.
- Each region carries exactly one primary feature type after overlap resolution via the accession ranking file.

## Limitations

- Transcript selection depends on GENCODE tag filtering (one best transcript per gene); alternative isoform structure is collapsed, so windows reflect only the chosen transcript.
- Skipping peak calling assumes annotated transcripts are available and adequate; multi-mapping/repetitive-element handling is separate from tiling.
- The overall Skipper analysis requires a matched input sample to build the beta-binomial model, so tiling alone is not sufficient for downstream binding tests.

## Evidence

- [other] Skipper partitions the transcriptome into windows of at most 100 nt by iterating over ranked features and transcript types, creating variable-length windows that never traverse exon-intron junctions or gene boundaries.: [evidence span withheld — anchor status: not_found]
