---
name: pipeline-runtime-measurement
description: Use when quantifying end-to-end runtime and output file sizes of a NEAT
  NGS analysis on the truncated ChIPseq/RNAseq test datasets, or when validating speed
  claims for a sequenced run assessment within 24 h.
when_to_use_negative:
- Input is full-scale (non-truncated) sequencing data where wall-time and storage
  expectations differ; use cluster-submission limits and customized storage settings
  instead of the truncated-dataset benchmark
- The goal is exploratory data analysis (metagenomic or differentially expressed gene
  analysis) rather than pipeline runtime and file-size characterization
- The operating environment is not Mac OSx or a qsub/bsub cluster, since NEAT support
  is documented only for these
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: NEAT
  role: framework whose runtime and output file sizes are benchmarked; launched via
    double-clickable executables or command line
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R package used by the NEAT 'QC' module executed within the timed pipeline
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: aligner invoked by the 'map' module for ChIPseq reads
- name: samtools
  role: filter module backend (view/sort/rmdup/index) producing the .bam outputs whose
    sizes are recorded
- name: MACS
  role: peak-calling algorithm selectable in the 'peakcalling' module
- name: SPP
  role: alternative peak-calling algorithm selectable in the 'peakcalling' module
- name: R
  role: runtime environment for QC, GRanges object generation and wiggle file creation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:ea332ed5a836d7944e42c295f9d9778dcab5e2e1b2ac98a0ab0ec317802613c5
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
  - bwa
  - samtools
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# pipeline-runtime-measurement

## Summary

Benchmarks NEAT's NGS pipeline execution time and output file-size footprint on the bundled truncated H3K4me3 ChIPseq and RNAseq test datasets (mouse embryonic stem cells), validating that the framework completes analyses well within the paper's <24-hour claim.

## When to use

Use when quantifying end-to-end runtime and output file sizes of a NEAT NGS analysis on the truncated ChIPseq/RNAseq test datasets, or when validating speed claims for a sequenced run assessment within 24 h.

## When NOT to use

- Input is full-scale (non-truncated) sequencing data where wall-time and storage expectations differ; use cluster-submission limits and customized storage settings instead of the truncated-dataset benchmark
- The goal is exploratory data analysis (metagenomic or differentially expressed gene analysis) rather than pipeline runtime and file-size characterization
- The operating environment is not Mac OSx or a qsub/bsub cluster, since NEAT support is documented only for these

## Inputs

- truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells (bundled with NEAT)
- truncated RNAseq test dataset from mouse embryonic stem cells (bundled with NEAT)
- NEAT Targets.txt with 'Steps_to_execute_pipe' blocks (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges)
- AdvancedSettings.txt filtering and peak-calling parameters

## Outputs

- neat_benchmark.csv summarizing runtime and file sizes per dataset
- .bam and .bai alignment files (bam ~4-6 Gb at full scale)
- GRanges objects (~40-60 Mb at full scale)
- .wig wiggle files (~4-6 Mb at full scale)

## How to apply

Obtain the truncated H3K4me3 ChIPseq and RNAseq test datasets bundled with NEAT from github.com/pschorderet/NEAT. Create a new project with the 'New Project' application, filling in the centralized Targets.txt file with the 'Steps_to_execute_pipe' blocks set to unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges. Launch the pipeline via the double-clickable executable (or command line on a qsub/bsub cluster); reads are mapped with bwa or bowtie (Tophat for RNAseq), filtered with samtools view/sort/rmdup/index, and peaks called with MACS or SPP, producing GRanges objects and wiggle files. Time the full execution per dataset, then record the sizes of the resulting .bam, GRanges and .wig files and compile them into a single summary table (neat_benchmark.csv). Validation relies on the paper's size regimes: wig files ~4-6 Mb (smallest), GRanges ~40-60 Mb, and bam ~4-6 Gb for full-scale data (largest), with truncated runs expected to finish in under two hours per dataset and full-scale analyses within 24 hours of sequencing-run completion.

## Related tools

- **NEAT** (framework whose runtime and output file sizes are benchmarked; launched via double-clickable executables or command line) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R package used by the NEAT 'QC' module executed within the timed pipeline) — https://github.com/tgirke/systemPipeR
- **bwa** (aligner invoked by the 'map' module for ChIPseq reads)
- **samtools** (filter module backend (view/sort/rmdup/index) producing the .bam outputs whose sizes are recorded)
- **MACS** (peak-calling algorithm selectable in the 'peakcalling' module)
- **SPP** (alternative peak-calling algorithm selectable in the 'peakcalling' module)
- **R** (runtime environment for QC, GRanges object generation and wiggle file creation)

## Examples

```
Steps_to_execute_pipe = unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges  # in NEAT Targets.txt, then time the double-clickable executable and save timings to neat_benchmark.csv
```

## Evaluation signals

- Truncated analysis completes in less than two hours per dataset
- Produced file classes match the paper's size regimes: .wig ~4-6 Mb (smallest), GRanges ~40-60 Mb, .bam ~4-6 Gb (largest) at full scale
- Runtime for full-scale data is under 24 hours, consistent with the paper's efficiency claim
- All 'Steps_to_execute_pipe' blocks (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) executed and outputs archived per project
- neat_benchmark.csv contains runtime and file-size entries for both the ChIPseq and RNAseq datasets

## Limitations

- Benchmark validity depends on truncated test data; size regimes (wig ~4-6 Mb, GRanges ~40-60 Mb, bam ~4-6 Gb) are reported for full-scale data and will not match truncated runs
- Storage space, maximum number of job submissions and wall time depend on the institution's cluster and must be customized
- NEAT is documented as running on Mac OSx; no Linux/Windows support is stated
- No NEAT version number or changelog is provided, so benchmark results may not be reproducible against a pinned release

## Evidence

- [abstract] Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive: "Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive"
- [methods] The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [methods] The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis: "The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [discussion] NEAT, including tutorials and test data, is publicly available on GitHub (https://github.com/pschorderet/NEAT).: "NEAT, including tutorials and test data, is publicly available on GitHub (https://github.com/pschorderet/NEAT)."
- [readme] NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications.: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
