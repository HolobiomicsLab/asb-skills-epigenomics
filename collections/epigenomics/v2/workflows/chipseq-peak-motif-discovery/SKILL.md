---
name: chipseq-peak-motif-discovery-workflow
description: 'Use when paired ChIP-seq immunoprecipitation and control FASTQ reads
  must be aligned and quality-assessed, converted into control-aware peaks, and analyzed
  for over-represented sequence motifs against an explicit matched background.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - fastq-read-alignment-quality-assessment
  - bwa-read-alignment-and-bam-processing
  - reference-genome-data-preparation
  - chipseq-quality-metric-computation
  - chip-seq-peak-calling-with-control
  - chipseq-peak-calling-with-macs
  - chipseq-peak-calling--2dd556ad6f61330a
  - macs2-peak-calling
  - discriminative-positive-negative-dataset-construction
  - chipseq-fasta-dataset-handling
  - chipseq-overrepresented-motif-discovery
  - dna-motif-overrepresentation-analysis
  - fisher-exact-test-motif-significance
  - meme-format-motif-output
  member_tools:
  - BWA
  - ChiLin
  - FastQC
  - MACS2
  - bedtools
  - samtools
  - seqtk
  - DGW
  - MACS
  - Python
  - DiNAMO
  - JASPAR
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/epigenomics/v2
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ChIP-seq Peak Calling and Discriminative Motif Discovery

## Summary

Align and quality-assess ChIP-seq reads, call control-aware peaks, construct matched signal and background FASTA files, and discover enriched transcription-factor motifs.


## When to use

Use when paired ChIP-seq immunoprecipitation and control FASTQ reads must be aligned and quality-assessed, converted into control-aware peaks, and analyzed for over-represented sequence motifs against an explicit matched background.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — align_and_assess_reads

**Goal:** align ChIP-seq treatment and control reads and report early alignment quality

**EDAM operation:** http://edamontology.org/operation_0292

**Inputs:** paired-fastq/chipseq-immunoprecipitation, paired-fastq/chipseq-control, directory/chilin-species-reference-bundle · **Outputs:** bam/aligned-chipseq-immunoprecipitation, bam/aligned-chipseq-control, qc-report/chipseq-read-alignment

**Candidate leaf skills:** `fastq-read-alignment-quality-assessment` (primary), `bwa-read-alignment-and-bam-processing`, `reference-genome-data-preparation`, `chipseq-quality-metric-computation`

**Tools (primary):** BWA, ChiLin, FastQC, MACS2, bedtools, samtools, seqtk

**Other candidate tools:** R

**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-016-1274-4

### Stage 2 — call_control_aware_peaks

**Goal:** call enriched ChIP-seq peaks against the matched control

**EDAM operation:** http://edamontology.org/operation_3222

**Inputs:** bam/aligned-chipseq-immunoprecipitation, bam/aligned-chipseq-control · **Outputs:** bed/control-aware-chipseq-peaks

**Candidate leaf skills:** `chip-seq-peak-calling-with-control` (primary), `chipseq-peak-calling-with-macs`, `chipseq-peak-calling--2dd556ad6f61330a`, `macs2-peak-calling`

**Tools (primary):** DGW, MACS, MACS2, Python

**Other candidate tools:** Bowtie, R, SAMtools, SeqCode, gawk, spikChIP, NEAT, SPP, BWA, ChiLin, bedClip, bedGraphToBigWig, bedtools, seqtk, wigCorrelate

**Grounding:** 4 KB(s); DOIs: 10.1038/s41598-021-98889-7, 10.1186/s12859-016-0902-3, 10.1186/s12859-016-1274-4, 10.1186/s12859-016-1306-0

### Stage 3 — construct_motif_fastas

**Goal:** construct peak-sequence and matched-background FASTA files

**EDAM operation:** http://edamontology.org/operation_0238

**Inputs:** bed/control-aware-chipseq-peaks, fasta/reference-genome, bed/matched-background-regions · **Outputs:** fasta/chipseq-peak-signal, fasta/chipseq-matched-control

**Candidate leaf skills:** `discriminative-positive-negative-dataset-construction` (primary), `chipseq-fasta-dataset-handling`

**Tools (primary):** DiNAMO

**Other candidate tools:** HOMER, MEME

**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-018-2215-1

### Stage 4 — discover_overrepresented_motifs

**Goal:** identify motifs enriched in ChIP-seq peak sequences over matched background

**EDAM operation:** http://edamontology.org/operation_0239

**Inputs:** fasta/chipseq-peak-signal, fasta/chipseq-matched-control, yaml/dinamo-motif-configuration, motif-matrix-collection/jaspar-frequency-matrices · **Outputs:** meme/overrepresented-chipseq-motifs, motif-table/jaspar-matched-chipseq-motifs

**Candidate leaf skills:** `chipseq-overrepresented-motif-discovery` (primary), `dna-motif-overrepresentation-analysis`, `fisher-exact-test-motif-significance`, `meme-format-motif-output`

**Tools (primary):** DiNAMO, JASPAR

**Other candidate tools:** MEME, RSAT

**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-018-2215-1

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
