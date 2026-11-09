---
name: ngs-pipeline-execution-benchmarking
description: Use when you need to measure runtime and output file sizes of an end-to-end
  NEAT NGS pipeline on truncated ChIPseq/RNAseq test data, or to validate that pipeline
  completion fits within sequencing-run turnaround limits.
when_to_use_negative:
- Input is full-scale (non-truncated) sequencing data — expect runtimes approaching
  the paper's <24h bound rather than the <2h truncated benchmark, and storage/job-submission
  limits must be negotiated with the cluster
- The goal is single-step analysis (e.g., alignment only) rather than timing an end-to-end
  unzip → QC → chiprx → map → filter → peakcalling → cleanfiles → granges pipeline
- Running on Windows or Linux — the article states the operating system is limited
  to Mac OSx
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
tools:
- name: NEAT
  role: Framework whose double-clickable executables run the full unzip→QC→chiprx→map→filter→peakcalling→cleanfiles→granges
    pipeline being benchmarked
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R package used by the NEAT QC module for quality control of the test datasets
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: Alignment tool invoked by the 'map' module for ChIPseq read mapping
- name: bowtie
  role: Alternative alignment tool invoked by the 'map' module
- name: Tophat
  role: Alignment tool used for the RNAseq test dataset
- name: samtools
  role: view/sort/rmdup/index used by the 'filter' module to produce the .bam files
    measured in the benchmark
- name: MACS
  role: Peak-calling algorithm option in the 'peakcalling' module
- name: SPP
  role: Alternative peak-calling algorithm option in the 'peakcalling' module
- name: R
  role: Runtime environment for systemPipeR-based QC and GRanges object generation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:61702bd626f21d7e5168e885873c0f40e0478c233aa8137311fc1c84080ff45f
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ngs-pipeline-execution-benchmarking

## Summary

Benchmarks the NEAT NGS pipeline by timing full execution of truncated H3K4me3 ChIPseq and RNAseq test datasets (mouse embryonic stem cells) and recording output file-size regimes (.bam, GRanges, .wig), validating that analyses complete well within the paper's <24h target.

## When to use

Use when you need to measure runtime and output file sizes of an end-to-end NEAT NGS pipeline on truncated ChIPseq/RNAseq test data, or to validate that pipeline completion fits within sequencing-run turnaround limits.

## When NOT to use

- Input is full-scale (non-truncated) sequencing data — expect runtimes approaching the paper's <24h bound rather than the <2h truncated benchmark, and storage/job-submission limits must be negotiated with the cluster
- The goal is single-step analysis (e.g., alignment only) rather than timing an end-to-end unzip → QC → chiprx → map → filter → peakcalling → cleanfiles → granges pipeline
- Running on Windows or Linux — the article states the operating system is limited to Mac OSx

## Inputs

- Truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells (FASTQ, bundled with NEAT)
- Truncated RNAseq test dataset from mouse embryonic stem cells (FASTQ, bundled with NEAT)
- Centralized Targets.txt file with sample names, replicates, conditions, antibodies, and pipeline settings
- AdvancedSettings.txt with filtering parameters (duplicate-read handling, fragment size min/max)

## Outputs

- neat_benchmark.csv summarizing runtime and file sizes per dataset
- .bam/.bai files reorganized by the cleanfiles module
- GRanges objects (~40-60 Mb at full scale) for downstream analysis
- Wiggle (.wig) files (~4-6 Mb at full scale) for genome-browser visualization
- Per-step QC and peak-calling outputs from systemPipeR, MACS/SPP

## How to apply

Obtain the truncated H3K4me3 ChIPseq and RNAseq test datasets bundled with NEAT from the GitHub repository. Create a new project via the 'New Project' application and fill in the centralized Targets.txt file, specifying the 'Steps_to_execute_pipe' blocks as unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges. Launch the pipeline with the double-clickable executable (or command line), which maps reads with bwa/bowtie (Tophat for RNAseq), filters with samtools view/sort/rmdup/index, calls peaks with MACS or SPP, and generates GRanges objects and wiggle files. Time the full execution per dataset and record the sizes of the .bam, GRanges, and .wig outputs; compile both datasets into a single summary table (neat_benchmark.csv). Validation: the truncated analysis should complete in less than two hours per dataset, and file classes should match the reported size regimes — wig ~4-6 Mb (smallest), GRanges ~40-60 Mb, bam ~4-6 Gb for full-scale data (largest).

## Related tools

- **NEAT** (Framework whose double-clickable executables run the full unzip→QC→chiprx→map→filter→peakcalling→cleanfiles→granges pipeline being benchmarked) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R package used by the NEAT QC module for quality control of the test datasets) — https://github.com/tgirke/systemPipeR
- **bwa** (Alignment tool invoked by the 'map' module for ChIPseq read mapping)
- **bowtie** (Alternative alignment tool invoked by the 'map' module)
- **Tophat** (Alignment tool used for the RNAseq test dataset)
- **samtools** (view/sort/rmdup/index used by the 'filter' module to produce the .bam files measured in the benchmark)
- **MACS** (Peak-calling algorithm option in the 'peakcalling' module)
- **SPP** (Alternative peak-calling algorithm option in the 'peakcalling' module)
- **R** (Runtime environment for systemPipeR-based QC and GRanges object generation)

## Examples

```
open NEAT 'New Project' application; fill Targets.txt with Steps_to_execute_pipe: unzip QC chiprx map filter peakcalling cleanfiles granges; launch double-clickable NEAT executable; record wall time and sizes of .bam, GRanges, .wig outputs per dataset into neat_benchmark.csv
```

## Evaluation signals

- Truncated analysis completes in less than two hours per dataset (well within the paper's <24h bound)
- Produced file classes match the reported size regimes: .wig ~4-6 Mb, GRanges ~40-60 Mb, .bam ~4-6 Gb for full-scale data
- .wig files are the smallest outputs and .bam files are the largest (ordering invariant)
- All pipeline steps in 'Steps_to_execute_pipe' (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) complete without error
- neat_benchmark.csv contains runtime and file-size rows for both the ChIPseq and RNAseq test datasets

## Limitations

- NEAT runs on Mac OSx only; no Linux/Windows support is stated in the article
- Runtime depends on cluster settings — storage space, maximum number of job submissions, wall time, and cluster-specific parameters must be customized since NEAT runs on the institution's cluster
- NEAT was still under development ('Devel mode') and no NEAT version number or changelog is stated; benchmarks may not be reproducible across repository states
- The file-size regimes are reported for full-scale data; truncated test datasets will produce proportionally smaller outputs, so only the relative ordering (wig < GRanges < bam) is directly checkable

## Evidence

- [abstract] Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive: "Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive"
- [methods] Truncated test datasets: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [methods] GRanges module output size: "The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis"
- [intro] Pipeline modules: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] Cluster customization: "Storage space, maximum number of job submissions, wall time and cluster-specific parameters can be customized as NEAT is run on the institution's cluster"
- [readme] Test data availability: "Novice users can follow through the entire analysis of a test dataset provided as part of the package. Please follow the tutorials found in the 'vignette' folder"
