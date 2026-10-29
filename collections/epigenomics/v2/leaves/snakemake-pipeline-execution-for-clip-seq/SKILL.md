---
name: snakemake-pipeline-execution-for-clip-seq
description: Use when raw eCLIP FASTQ files with matched IP and input samples must
  be converted into reproducible enriched binding-site windows transcriptome-wide,
  replacing per-tool peak calling with an integrated Snakemake workflow.
when_to_use_negative:
- CLIP datasets lacking a matched input sample — Skipper needs the input to construct
  the beta-binomial model for statistical testing and cannot analyze such data
- Only pre-computed peak calls or feature tables are available and no raw reads —
  this skill operates end-to-end from unprocessed reads
- A single small machine with no HPC is the only compute option and the dataset is
  large — Skipper is primarily designed for HPC clusters where jobs are parallelized
  across compute nodes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_0659
tools:
- name: Skipper
  role: End-to-end Snakemake workflow converting unprocessed eCLIP reads into annotated
    binding sites via window tiling over transcripts
  repo: github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine that orchestrates Skipper stages and auto-installs conda environments
    via snakeconda
- name: STAR
  role: Aligns preprocessed eCLIP reads using a genome index built from the GENCODE
    GFF and genome FASTA
- name: snakemake-executor-plugin-slurm 1.4.0
  role: SLURM cluster executor plugin for running Skipper stages across HPC compute
    nodes
- name: Miniconda/conda
  role: Environment manager used to install Snakemake and the SLURM executor plugin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:dd81c1109085e2278deafbc5a18b26305dfd9dab7a3143070323a177637b52eb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - CLIPper
  - Miniconda/conda
  - Piranha
  - STAR
  - Skipper
  - Snakemake
  - Snakemake 9.12.0
  - snakemake-executor-plugin-slurm 1.4.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# snakemake-pipeline-execution-for-clip-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the Skipper end-to-end eCLIP analysis pipeline (Snakemake-based) that converts unprocessed reads into annotated binding sites via preprocessing/trimming, alignment against a STAR genome index built from GENCODE GFF and genome FASTA, GFF partitioning, enriched window identification, fine-mapping, and motif analysis.

## When to use

Use when raw eCLIP FASTQ files with matched IP and input samples must be converted into reproducible enriched binding-site windows transcriptome-wide, replacing per-tool peak calling with an integrated Snakemake workflow.

## When NOT to use

- CLIP datasets lacking a matched input sample — Skipper needs the input to construct the beta-binomial model for statistical testing and cannot analyze such data
- Only pre-computed peak calls or feature tables are available and no raw reads — this skill operates end-to-end from unprocessed reads
- A single small machine with no HPC is the only compute option and the dataset is large — Skipper is primarily designed for HPC clusters where jobs are parallelized across compute nodes

## Inputs

- Unprocessed eCLIP FASTQ reads for matched IP and input samples
- Sample manifest CSV (e.g. Example_manifest.csv) with absolute sample paths
- Skipper config YAML (e.g. Example_config.yaml) with absolute paths
- GENCODE/Ensembl GFF3 annotation (e.g. gencode.v49.basic.annotation.gff3), pre-filtered for best transcript per gene
- GENCODE genome FASTA (e.g. GRCh38.primary_assembly.genome.fa)
- Optional: user-defined blacklist of reproducible regions and UCSC Table Browser repeat-element coordinates

## Outputs

- Reproducible enriched binding-site windows per RBP
- Feature-annotated binding sites (introns, exons, UTRs, repetitive elements)
- Fine-mapped window results and motif analysis outputs
- STAR genome index and parsed GFF generated on first run

## How to apply

Clone github.com/YeoLab/skipper and create a Snakemake 9.12.0 conda environment (plus snakemake-executor-plugin-slurm=1.4.0 for HPC execution). Download the GRCh38.primary_assembly.genome.fa (GENCODE release 49) and a GFF such as gencode.v49.basic.annotation.gff3, then pre-filter the GFF: select one best transcript per gene using the 'tag' column, run gff_expression_filter.py, and resolve overlapping feature annotations so windows do not cross intron/exon boundaries. Edit the example config YAML (absolute paths to the Skipper directory and output location) and the manifest CSV listing samples, then launch 'snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm'. The first run is slow because snakeconda builds all environments, parses the GFF, and generates the STAR genome index; subsequent runs are much faster. On first-run issues, work from an interactive node with ample RAM (e.g. 16G) and consult the troubleshooting section.

## Related tools

- **Skipper** (End-to-end Snakemake workflow converting unprocessed eCLIP reads into annotated binding sites via window tiling over transcripts) — github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine that orchestrates Skipper stages and auto-installs conda environments via snakeconda)
- **STAR** (Aligns preprocessed eCLIP reads using a genome index built from the GENCODE GFF and genome FASTA)
- **snakemake-executor-plugin-slurm 1.4.0** (SLURM cluster executor plugin for running Skipper stages across HPC compute nodes)
- **Miniconda/conda** (Environment manager used to install Snakemake and the SLURM executor plugin)

## Examples

```
conda activate snakemake9; snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The first run completes without errors through GFF parsing and STAR index generation; subsequent Skipper runs are markedly faster
- Output contains reproducible enriched windows annotated against transcript features (introns, exons, UTRs), avoiding windows that cross intron/exon boundaries
- When benchmarked on ENCODE eCLIP, aggregate counts match reported values: average ~21,310 enriched windows vs Piranha ~5,039 and CLIPper ~6,904, with Skipper exceeding existing methods for ~72% of RBPs
- Snakemake logs show all pipeline stages ran (preprocessing/trimming, alignment, GFF partitioning, enriched window identification, fine-mapping, motif analysis) and SLURM job logs are written to the configured log directory
- Binding to annotated repetitive elements is called, consistent with Skipper identifying bound elements for 99% of eCLIP experiments

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets without such controls cannot be analyzed
- The first Skipper run is very slow because all conda environments (snakeconda), GFF parsing, and STAR index generation must be completed, and can consume substantial RAM
- Users may blacklist reproducible regions, but best practices for blacklisting are not yet established
- Post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data

## Evidence
