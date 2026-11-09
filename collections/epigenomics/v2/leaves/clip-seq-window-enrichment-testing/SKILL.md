---
name: clip-seq-window-enrichment-testing
description: Use when analyzing eCLIP/CLIP-seq data to detect RNA-binding protein
  binding sites transcriptome-wide, especially when peak callers underperform or when
  binding to repetitive elements or low-abundance transcripts must be tested.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample — Skipper requires matched input to
  construct the beta-binomial model and cannot analyze such data
- The goal is motif discovery or binding affinity estimation directly from CLIP data
  — post hoc fine-mapping does not learn these directly
- Raw, unfiltered GFF annotations with many alternative isoform transcripts are supplied
  without selecting one best transcript per gene, compromising window interpretability
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0659
tools:
- name: Skipper
  role: End-to-end Snakemake workflow that tiles windows over annotated transcripts
    and performs the GC-stratified beta-binomial enrichment testing; learns overdispersion
    and GC bias separately for transcriptomic windows and repetitive elements
  repo: github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper pipeline, optionally with the
    snakemake-executor-plugin-slurm=1.4.0 on HPC clusters
- name: VGAM
  role: R package providing the beta-binomial family used to fit per-GC-decile overdispersion
    rho and compute enrichment p-values
- name: STAR
  role: Aligner used to generate the genome index and align preprocessed eCLIP reads
    upstream of window counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:e5f7a6e8233dfdd5c11c6453fef739c1d42dd4dcad58311993021d50b057da16
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - STAR
  - Skipper
  - Snakemake 9.12.0
  - VGAM
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clip-seq-window-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically tests tiled transcriptomic windows and repetitive elements for enrichment of IP read-start signal over matched input using a GC-stratified beta-binomial model, skipping peak calling entirely. This is Skipper's core statistical framework for calling RBP binding sites from eCLIP data.

## When to use

Use when analyzing eCLIP/CLIP-seq data to detect RNA-binding protein binding sites transcriptome-wide, especially when peak callers underperform or when binding to repetitive elements or low-abundance transcripts must be tested.

## When NOT to use

- The CLIP dataset lacks a matched input sample — Skipper requires matched input to construct the beta-binomial model and cannot analyze such data
- The goal is motif discovery or binding affinity estimation directly from CLIP data — post hoc fine-mapping does not learn these directly
- Raw, unfiltered GFF annotations with many alternative isoform transcripts are supplied without selecting one best transcript per gene, compromising window interpretability

## Inputs

- Per-window read-start count tables for input and CLIP/IP fractions across tiled transcriptomic windows
- Per-window counts over annotated repetitive elements (UCSC Table Browser repeat coordinates)
- Filtered annotation GFF (e.g., gencode.v49.basic.annotation.gff3), one best transcript per gene
- Alignment BAMs (e.g., STAR-aligned) for CLIP replicates and matched input samples
- Skipper config YAML and sample manifest CSV

## Outputs

- Genome-wide table of window coordinates, statistics, and per-window enrichment p-values
- Reproducible enriched window calls (significant in the DEFINITION_OF_REPRODUCIBILITY number of CLIP replicates)
- Beta-binomial fits (GC-decile-specific rho and mu) for transcriptomic windows and repetitive elements

## How to apply

Tile windows over annotated transcripts (from a GENCODE or Ensembl GFF, pre-filtered to one best transcript per gene via the 'tag' column), and separately over repetitive element coordinates from the UCSC Table Browser. Load per-window read-start counts for input and IP fractions and drop 'skyscraper' windows flagged by the GINI_CUTOFF filter (default ~0.9; stricter 0.8, more lenient 0.95). Stratify surviving windows into deciles by GC content and, within each decile, fit a beta-binomial model (e.g., VGAM beta-binomial family in R) estimating overdispersion rho — preferably from multiple input replicates (OVERDISPERSION_MODE='input'). Apply decile-specific rho and mu to compute per-window IP-vs-input enrichment p-values, excluding windows with fewer than THRESHOLD_MIN total reads, then mark windows significant in at least the DEFINITION_OF_REPRODUCIBILITY number of CLIP replicates (default 'ALL', minimum 2) as reproducible. The rationale is that GC stratification and learned overdispersion correct read-count biases that peak callers handle poorly.

## Related tools

- **Skipper** (End-to-end Snakemake workflow that tiles windows over annotated transcripts and performs the GC-stratified beta-binomial enrichment testing; learns overdispersion and GC bias separately for transcriptomic windows and repetitive elements) — github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper pipeline, optionally with the snakemake-executor-plugin-slurm=1.4.0 on HPC clusters)
- **VGAM** (R package providing the beta-binomial family used to fit per-GC-decile overdispersion rho and compute enrichment p-values)
- **STAR** (Aligner used to generate the genome index and align preprocessed eCLIP reads upstream of window counting)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file exists and contains a p-value column for every window, with all p-values within [0, 1]
- Number of retained windows is consistent with the configured THRESHOLD_MIN and GINI_CUTOFF filters
- Reproducible windows are flagged in at least the DEFINITION_OF_REPRODUCIBILITY number of CLIP replicates (minimum 2, default 'ALL')
- GC-stratified fits cover all GC deciles, with overdispersion estimated from input replicates (OVERDISPERSION_MODE='input')
- Both transcriptomic windows and repetitive elements are tested — bound repetitive elements should be identified for ~99% of eCLIP experiments as reported

## Limitations

- Skipper requires a matched input sample; CLIP datasets lacking these controls cannot be analyzed
- Post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data
- Best practices for blacklisting reproducible regions are not yet established
- The gapped k-mer SVM used downstream for motif modeling is less sophisticated than emerging deep learning tools

## Evidence

- [results] Skipper learns overdispersion in read counts and GC bias separately for transcriptomic windows and repetitive elements, then computes p values from the fit beta-binomial distributions to identify enriched windows.: [evidence span withheld — anchor status: not_found]
