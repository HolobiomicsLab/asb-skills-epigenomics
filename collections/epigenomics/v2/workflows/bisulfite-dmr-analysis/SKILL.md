---
name: bisulfite-dmr-analysis-workflow
description: 'Use when paired-end whole-genome bisulfite sequencing reads must be
  mapped, reduced to six-column per-CpG methylated and unmethylated count tables,
  aggregated into regional total and methylated counts, and tested between externally
  defined groups.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - bisulfite-read-alignment--c6ac0ff26c82f263
  - wgbs-read-alignment-bisulfite-mode
  - bowtie2-based-short-read-alignment
  - bisulfite-read-alignment--a2d18d44f25cd691
  - cpg-methylation-count-extraction
  - strand-merged-methylation-calling
  - bismark-sam-alignment-parsing
  - methylc-seq-alignment-processing
  - cpg-site-region-clustering
  - pairwise-group-methylation-comparison
  - reference-implementation-output-validation
  - beta-binomial-differential-methylation-testing
  - dss-wald-test-workflow
  - methylc-seq-region-dmr-identification
  member_tools:
  - Bismark
  - Bowtie2
  - DMRfinder
  - DSS
  - Python
  - R
  - Samtools
  - bsseq
  - findDMRs.r
  - DMRfinder (extract_CpG_data.py)
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

# Whole-Genome Bisulfite Differential Methylation Analysis

## Summary

Align bisulfite reads, extract per-CpG methylated and unmethylated counts, sum nearby CpGs into per-sample regional totals, and test regions with a beta-binomial model.


## When to use

Use when paired-end whole-genome bisulfite sequencing reads must be mapped, reduced to six-column per-CpG methylated and unmethylated count tables, aggregated into regional total and methylated counts, and tested between externally defined groups.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — align_bisulfite_reads

**Goal:** align bisulfite-treated reads to the reference genome

**EDAM operation:** http://edamontology.org/operation_3198

**Inputs:** paired-fastq/whole-genome-bisulfite, fasta/reference-genome · **Outputs:** bam/bismark-bisulfite-alignment

**Candidate leaf skills:** `bisulfite-read-alignment--c6ac0ff26c82f263` (primary), `wgbs-read-alignment-bisulfite-mode`, `bowtie2-based-short-read-alignment`, `bisulfite-read-alignment--a2d18d44f25cd691`

**Tools (primary):** Bismark, Bowtie2, DMRfinder, DSS, Python, R, Samtools, bsseq, findDMRs.r

**Other candidate tools:** Msuite, Mviewer, build.index.sh, Apache Spark, BiSpark, Bison, CloudAligner, Cutadapt, Sherman

**Grounding:** 3 KB(s); DOIs: 10.1016/j.patter.2020.100127, 10.1186/s12859-017-1909-0, 10.1186/s12859-018-2498-2

### Stage 2 — extract_cpg_counts

**Goal:** extract methylated and unmethylated read counts at individual CpG sites

**EDAM operation:** http://edamontology.org/operation_3207

**Inputs:** bam/bismark-bisulfite-alignment · **Outputs:** cov/per-cpg-methylated-unmethylated-counts

**Candidate leaf skills:** `cpg-methylation-count-extraction` (primary), `strand-merged-methylation-calling`, `bismark-sam-alignment-parsing`, `methylc-seq-alignment-processing`

**Tools (primary):** Bismark, Bowtie2, DMRfinder, DMRfinder (extract_CpG_data.py), DSS, Python, R, Samtools, bsseq, findDMRs.r


**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-017-1909-0

### Stage 3 — cluster_cpg_regions

**Goal:** aggregate CpGs into regional total and methylated counts for each sample

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** cov/per-cpg-methylated-unmethylated-counts · **Outputs:** csv/clustered-region-total-and-methylated-counts

**Candidate leaf skills:** `cpg-site-region-clustering` (primary), `pairwise-group-methylation-comparison`, `reference-implementation-output-validation`

**Tools (primary):** Bismark, Bowtie2, DMRfinder, DSS, Python, R, bsseq, findDMRs.r


**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-017-1909-0

### Stage 4 — call_differentially_methylated_regions

**Goal:** test clustered regions for differential methylation between sample groups

**EDAM operation:** http://edamontology.org/operation_3206

**Inputs:** csv/clustered-region-total-and-methylated-counts, tsv/sample-group-design, yaml/dmr-filter-configuration · **Outputs:** csv/dmr-effects-and-pvalues

**Candidate leaf skills:** `beta-binomial-differential-methylation-testing` (primary), `dss-wald-test-workflow`, `methylc-seq-region-dmr-identification`

**Tools (primary):** DMRfinder, DSS, R, findDMRs.r


**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-017-1909-0

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
