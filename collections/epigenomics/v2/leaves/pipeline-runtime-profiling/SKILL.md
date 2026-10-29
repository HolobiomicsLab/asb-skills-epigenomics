---
name: pipeline-runtime-profiling
description: Use when choosing between read-trimming or read-mapping tools for an
  NGS pipeline and a reproducible comparison of wall-clock runtime plus retained-read
  yield/mapping accuracy is needed to justify the selection.
when_to_use_negative:
- The goal is to execute a production pipeline rather than benchmark tool performance
  — CIPHER currently only runs entire workflows, not subsets of tools, so timing individual
  tools must be done outside CIPHER
- The input is already trimmed, aligned, or otherwise pre-processed — runtime profiling
  requires a common raw FASTQ starting point for a fair comparison
- A tool has already been selected and no comparative runtime/accuracy justification
  is required
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3172
tools:
- name: CIPHER
  role: Workflow platform whose default trimming (BBDuk with mapq <20 / length <10
    filtering) and mapping (BBMap) selections and --download_data reference fetch
    are being benchmarked
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: BBDuk
  role: Fastest trimmer benchmarked; candidate best choice for read trimming and adapter
    removal
- name: Trimmomatic
  role: Competing trimmer benchmarked for retained-read yield and runtime
- name: cutadapt
  role: Competing trimmer benchmarked for retained-read yield and runtime
- name: BBMap
  role: Splice-unaware aligner benchmarked; CIPHER's default mapping tool
- name: BWA
  role: Splice-unaware aligner benchmarked for runtime and mapping statistics
- name: Bowtie2
  role: Splice-unaware aligner benchmarked for runtime and mapping statistics
- name: STAR
  role: Splice-aware aligner included for completeness in runtime and mapping-statistics
    comparison
- name: HISAT2
  role: Splice-aware aligner included for completeness in runtime and mapping-statistics
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:1b6e83742c4614be7ebbf47c029a40c409eed09b18fac1c72496079de24fbf3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - BBDuk
  - BBMap
  - BWA
  - Bowtie2
  - CIPHER
  - FastQC
  - HISAT2
  - STAR
  - Trimmomatic
  - cutadapt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# pipeline-runtime-profiling

## Summary

Benchmark the wall-clock runtime of NGS pre-processing tools — read trimmers (Trimmomatic, cutadapt, BBDuk) and aligners (BWA, Bowtie2, BBMap, STAR, HISAT2) — on identical public FASTQ input, recording runtime alongside accuracy metrics to justify tool selection as CIPHER did.

## When to use

Use when choosing between read-trimming or read-mapping tools for an NGS pipeline and a reproducible comparison of wall-clock runtime plus retained-read yield/mapping accuracy is needed to justify the selection.

## When NOT to use

- The goal is to execute a production pipeline rather than benchmark tool performance — CIPHER currently only runs entire workflows, not subsets of tools, so timing individual tools must be done outside CIPHER
- The input is already trimmed, aligned, or otherwise pre-processed — runtime profiling requires a common raw FASTQ starting point for a fair comparison
- A tool has already been selected and no comparative runtime/accuracy justification is required

## Inputs

- Public raw FASTQ file (e.g. ENCODE ChIP-seq H3K4me1 HCT116 sample, single- or pair-ended)
- Matching Ensembl/Gencode reference genome FASTA
- Matching Gencode GTF annotation file
- Installed trimming tools (Trimmomatic, cutadapt, BBDuk) and aligners (BWA, Bowtie2, BBMap, STAR, HISAT2)

## Outputs

- benchmark_metrics.csv with one metrics row per tool (8 tools), containing retained-read yield, unique/multi-mapped/low-quality read counts, and wall-clock runtime
- Per-tool aligner statistics parse of unique, multi-mapped, and low-quality/unmapped read counts

## How to apply

Start from a public raw FASTQ dataset (e.g. an ENCODE ChIP-seq H3K4me1 HCT116 sample) with matching Ensembl/Gencode FASTA and GTF references fetched via CIPHER's --download_data flag. Trim the same FASTQ with Trimmomatic, cutadapt, and BBDuk (CIPHER's default filtering removes reads with mapq <20 and length <10), recording retained-read yield after each tool. Map the trimmed reads with splice-unaware aligners (BWA, Bowtie2, BBMap) and splice-aware aligners (STAR, HISAT2), running mappers in Teaser's default mode with no additional parameters, and parse each aligner's statistics for unique, multi-mapped, and low-quality/unmapped read counts. Record wall-clock runtime per tool (use CIPHER's baseline of six ~30-million-read ChIP-seq datasets processed in 7 h 23 min as a throughput reference point). Compile trimming yield, unique/multi-mapped/low-quality counts, and runtime for all 8 tools into benchmark_metrics.csv; the skill is validated when BBDuk is identified as the fastest, most accurate trimmer and BBMap as the default aligner, matching CIPHER's reported selections.

## Related tools

- **CIPHER** (Workflow platform whose default trimming (BBDuk with mapq <20 / length <10 filtering) and mapping (BBMap) selections and --download_data reference fetch are being benchmarked) — https://github.com/c-guzman/cipher-workflow-platform
- **BBDuk** (Fastest trimmer benchmarked; candidate best choice for read trimming and adapter removal)
- **Trimmomatic** (Competing trimmer benchmarked for retained-read yield and runtime)
- **cutadapt** (Competing trimmer benchmarked for retained-read yield and runtime)
- **BBMap** (Splice-unaware aligner benchmarked; CIPHER's default mapping tool)
- **BWA** (Splice-unaware aligner benchmarked for runtime and mapping statistics)
- **Bowtie2** (Splice-unaware aligner benchmarked for runtime and mapping statistics)
- **STAR** (Splice-aware aligner included for completeness in runtime and mapping-statistics comparison)
- **HISAT2** (Splice-aware aligner included for completeness in runtime and mapping-statistics comparison)

## Examples

```
nextflow run cipher.nf -with-singularity <cipher.img> --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH>
```

## Evaluation signals

- benchmark_metrics.csv contains one metrics row for each of the 8 tools (Trimmomatic, cutadapt, BBDuk, BWA, Bowtie2, BBMap, STAR, HISAT2)
- The best-performing trimmer is BBDuk (fastest runtime combined with accuracy, ease of use, and single-/pair-end support), reproducing CIPHER's selection
- BBMap reproduces CIPHER's reported default-aligner selection
- Each tool row carries a recorded wall-clock runtime, a retained-read yield (for trimmers), and unique/multi-mapped/low-quality read counts (for aligners)
- All mappers were run under Teaser's default mode with no additional parameters, keeping comparisons controlled

## Limitations

- Software version numbers for the tested tools were not recorded in the source text, complicating exact reproduction
- All mappers were run in Teaser's default mode with no additional parameters, so results may not reflect tool-specific tuned performance
- CIPHER currently only runs entire workflows, not subsets of tools, so per-tool timing cannot be obtained from CIPHER itself
- Timing depends on machine configuration; CIPHER's own baseline (six ~30-million-read ChIP-seq datasets in 7 h 23 min) was obtained on a specific setup and parallelization (Nextflow, Singularity containers)

## Evidence

- [results] CIPHER baseline throughput: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [results] mapper benchmarking conditions: "All mappers were run in Teaser’s default mode with no additional parameters unless otherwise indicated."
- [results] mapper set tested: "DNA mappers (BBMap, BWA, and Bowtie2) using ENCODE datasets"
- [readme] parallelization/runtime control: [evidence span withheld — anchor status: external_doc]
