---
name: reproducible-enriched-window-calling
description: Use when analyzing eCLIP/CLIP-seq replicates to detect RBP binding transcriptome-wide
  as reproducible enriched windows (with annotation metadata) rather than peaks, especially
  when repetitive elements or diverse binding modes break peak callers.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample, which Skipper needs to construct
  the beta-binomial model for statistical testing
- The goal is not window-level binding detection but a downstream task already covered,
  e.g., feature tables for benchmarking precision/recall/F1 against truth sets
- The annotation GFF has not been filtered to one best transcript per gene, which
  Skipper requires for interpretable window selection
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline producing reproducible enriched windows; run via
    Snakemake with STAR alignment
  repo: https://github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper pipeline steps
- name: snakemake-executor-plugin-slurm 1.4.0
  role: SLURM executor plugin for running Skipper jobs in parallel across HPC compute
    nodes
- name: STAR
  role: Aligner used to generate the genome index and align trimmed eCLIP reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:8b1deb78cf363b51ac5c211c2fc2a5be6247b9b2ca8229c0776d082a739e6185
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
  - STAR
  - Skipper
  - Snakemake 9.12.0
  - conda
  - snakemake-executor-plugin-slurm 1.4.0
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reproducible-enriched-window-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Skipper, an end-to-end Snakemake workflow, to convert raw eCLIP FASTQ reads into reproducible enriched windows over annotated transcripts, replacing conventional peak calling with a beta-binomial test of read-start signal in tiled fixed windows.

## When to use

Use when analyzing eCLIP/CLIP-seq replicates to detect RBP binding transcriptome-wide as reproducible enriched windows (with annotation metadata) rather than peaks, especially when repetitive elements or diverse binding modes break peak callers.

## When NOT to use

- The CLIP dataset lacks a matched input sample, which Skipper needs to construct the beta-binomial model for statistical testing
- The goal is not window-level binding detection but a downstream task already covered, e.g., feature tables for benchmarking precision/recall/F1 against truth sets
- The annotation GFF has not been filtered to one best transcript per gene, which Skipper requires for interpretable window selection

## Inputs

- Raw eCLIP FASTQ reads for CLIP and matched input (size-matched control) samples, per replicate
- Filtered annotation GFF file (e.g., gencode.v49.basic.annotation.gff3 from GENCODE)
- Reference genome FASTA (e.g., GRCh38.primary_assembly.genome.fa)
- Skipper config YAML file and sample manifest CSV
- Repeat-element coordinates from UCSC Table Browser (for repetitive-element annotation)

## Outputs

- reproducible_enriched_windows tables per RBP with annotated binding sites
- Fine-mapped window/called-site results
- Motif analysis results from the gapped k-mer SVM step

## How to apply

Set up a conda environment with Snakemake 9.12.0 and, for HPC runs, the snakemake-executor-plugin-slurm 1.4.0 plugin, then clone github.com/YeoLab/skipper and run the pipeline via a config YAML plus a sample manifest CSV. Skipper executes preprocessing and trimming, STAR alignment against a GENCODE/Ensembl genome (e.g., GRCh38.primary_assembly.genome.fa from GENCODE release 49), GFF partitioning (using a filtered GFF such as gencode.v49.basic.annotation.gff3 where the best transcript per gene is selected via the 'tag' column), enriched window identification, fine-mapping, and motif analysis. Statistical testing uses a beta-binomial model built from the matched input sample, testing read-start signal in fixed windows tiled over annotated transcripts while avoiding windows that cross intron/exon boundaries; parameters like GINI_CUTOFF remove 'skyscraper' windows with narrow peaks. Users may blacklist reproducible enriched windows of their choosing, and the ENCODE4 single-end protocol with at least 2 CLIP replicates per sample yields reproducible_enriched_windows tables as the primary output.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline producing reproducible enriched windows; run via Snakemake with STAR alignment) — https://github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper pipeline steps)
- **snakemake-executor-plugin-slurm 1.4.0** (SLURM executor plugin for running Skipper jobs in parallel across HPC compute nodes)
- **STAR** (Aligner used to generate the genome index and align trimmed eCLIP reads)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The run completes all five stages listed in the README: preprocessing/trimming, alignment, GFF partitioning, enriched window identification, fine-mapping (and motif analysis)
- reproducible_enriched_windows tables are produced for every RBP with at least 2 CLIP replicates per sample
- Binding-site counts are annotated against transcript features (introns, exons, UTRs, repetitive elements) rather than unannotated peak coordinates
- No windows cross intron/exon boundaries in the GFF-partitioned window set, and skyscraper windows failing the GINI_CUTOFF are removed
- Output site counts are substantially larger than peak callers (Skipper calls 210%-320% more transcriptomic binding sites on average, sometimes >1,000% more)

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Users can blacklist reproducible regions of their choosing, but best practices for blacklisting are not established
- The first run is slow because it must set up all conda environments (snakeconda), parse the GFF, and generate the STAR genome index
- Post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data

## Evidence
