---
name: bam-granges-wiggle-file-size-profiling
description: Use when reproducing or benchmarking an NGS pipeline's runtime and output
  file-size footprint on truncated ChIPseq/RNAseq test data, or validating that produced
  bam/GRanges/wig files match expected size regimes.
when_to_use_negative:
- Input is full-scale (non-truncated) sequencing data where the <2h per dataset validation
  criterion does not apply — the paper's own threshold is <24h at full scale
- The goal is analyzing the biological content of peaks or differential expression
  rather than profiling pipeline runtime and output file sizes
- Running on Linux or Windows — the article states the operating system is limited
  to Mac OSx
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: NEAT
  role: pipeline framework executed via double-clickable executable to run unzip/QC/chiprx/map/filter/peakcalling/cleanfiles/granges
    modules on test datasets
  repo: https://github.com/pschorderet/NEAT
- name: bwa
  role: aligner used by the NEAT 'map' module for ChIPseq reads
- name: bowtie
  role: alternative aligner used by the NEAT 'map' module
- name: Tophat
  role: aligner used for the RNAseq test dataset
- name: samtools
  role: view/sort/rmdup/index operations used by the NEAT 'filter' module to produce
    .bam files
- name: MACS
  role: peak-calling algorithm selectable in the 'peakcalling' module
- name: SPP
  role: alternative peak-calling algorithm selectable in the 'peakcalling' module
- name: systemPipeR
  role: R package underlying the NEAT 'QC' module
  repo: https://github.com/tgirke/systemPipeR
- name: R
  role: runtime environment for QC, GRanges object creation and downstream analysis
    modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:33c15578b17b1ac0eb0c681871fae6f524ee7688b2b1ab34117ba13288e703c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - MACS
  - NEAT
  - R
  - SPP
  - Tophat
  - bowtie
  - bwa
  - samtools
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bam-granges-wiggle-file-size-profiling

## Summary

Benchmarks NEAT NGS pipeline outputs by timing full execution on truncated test datasets and recording the sizes of .bam, GRanges and .wig files, validating that output size regimes (wig smallest ~4-6 Mb, GRanges ~40-60 Mb, bam largest ~4-6 Gb for full-scale data) match the published report.

## When to use

Use when reproducing or benchmarking an NGS pipeline's runtime and output file-size footprint on truncated ChIPseq/RNAseq test data, or validating that produced bam/GRanges/wig files match expected size regimes.

## When NOT to use

- Input is full-scale (non-truncated) sequencing data where the <2h per dataset validation criterion does not apply — the paper's own threshold is <24h at full scale
- The goal is analyzing the biological content of peaks or differential expression rather than profiling pipeline runtime and output file sizes
- Running on Linux or Windows — the article states the operating system is limited to Mac OSx

## Inputs

- truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells (bundled with NEAT)
- truncated RNAseq test dataset from mouse embryonic stem cells (bundled with NEAT)
- Targets.txt centralized project file with 'Steps_to_execute_pipe' blocks

## Outputs

- .bam (and .bai) files per test dataset
- GRanges objects per test dataset
- .wig files per test dataset
- neat_benchmark.csv summary table of runtime and file sizes per dataset

## How to apply

Obtain the truncated H3K4me3 ChIPseq and RNAseq test datasets (mouse embryonic stem cells) bundled with NEAT from its GitHub repository. Create a NEAT project with the 'New Project' application and populate the centralized Targets.txt file, setting the 'Steps_to_execute_pipe' blocks to unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges. Launch via the double-clickable executable (or command line) so that reads are mapped with bwa/bowtie (Tophat for RNAseq), filtered with samtools view/sort/rmdup/index, peaks called with MACS or SPP, and GRanges objects and wiggle files generated. Time the full execution per dataset — the validation target is under two hours per truncated dataset, consistent with the paper's claim that NEAT completes analyses in <24 hours at full scale. Record the sizes of .bam, GRanges and .wig outputs per dataset and check that wig files are the smallest and bam files the largest, matching the paper's size regimes (wig ~4-6 Mb; GRanges ~40-60 Mb; bam ~4-6 Gb for full-scale data). Compile results into a single summary table saved as neat_benchmark.csv.

## Related tools

- **NEAT** (pipeline framework executed via double-clickable executable to run unzip/QC/chiprx/map/filter/peakcalling/cleanfiles/granges modules on test datasets) — https://github.com/pschorderet/NEAT
- **bwa** (aligner used by the NEAT 'map' module for ChIPseq reads)
- **bowtie** (alternative aligner used by the NEAT 'map' module)
- **Tophat** (aligner used for the RNAseq test dataset)
- **samtools** (view/sort/rmdup/index operations used by the NEAT 'filter' module to produce .bam files)
- **MACS** (peak-calling algorithm selectable in the 'peakcalling' module)
- **SPP** (alternative peak-calling algorithm selectable in the 'peakcalling' module)
- **systemPipeR** (R package underlying the NEAT 'QC' module) — https://github.com/tgirke/systemPipeR
- **R** (runtime environment for QC, GRanges object creation and downstream analysis modules)

## Examples

```
NEAT --project mESC_H3K4me3 --Steps_to_execute_pipe unzip,QC,chiprx,map,filter,peakcalling,cleanfiles,granges && time NEAT --run
```

## Evaluation signals

- Truncated analysis completes in less than two hours per dataset
- File-class size ordering holds: .wig files are the smallest and .bam files the largest for each dataset
- Full-scale size regimes match the paper: wig ~4-6 Mb, GRanges ~40-60 Mb, bam ~4-6 Gb
- Output file classes (.bam, GRanges, .wig) match the modules executed in Steps_to_execute_pipe (unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges)
- Measurements for both datasets are compiled into a single neat_benchmark.csv summary table

## Limitations

- Validation thresholds (<2 h per truncated dataset) apply only to the bundled truncated test data; full-scale data is bounded by the paper's <24 h claim
- Size regimes reported in the paper correspond to full-scale data; truncated datasets will produce proportionally smaller files
- NEAT runs only on Mac OSx per the article; no Linux/Windows support is stated, and no NEAT version number or changelog is provided

## Evidence

- [abstract] Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive: "Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive"
- [methods] truncated test datasets: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [methods] GRanges module output sizes: "The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis"
- [abstract] small-sized output files: "the small-sized files produced by NEAT allow users to easily manipulate, consolidate and share datasets from different users and institutions"
- [intro] NEAT automated modules: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
