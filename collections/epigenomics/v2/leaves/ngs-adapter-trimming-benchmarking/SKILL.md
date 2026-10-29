---
name: ngs-adapter-trimming-benchmarking
description: Use when choosing between NGS read-trimming or alignment tools (e.g.,
  BBDuk vs cutadapt vs Trimmomatic; BBMap vs BWA vs STAR) and you need speed-vs-accuracy
  metrics on a common ENCODE FASTQ input.
when_to_use_negative:
- Input is already trimmed, adapter-free FASTQ where only mapping benchmarking is
  needed
- The goal is routine production trimming/alignment rather than a tool-selection benchmark
  (use CIPHER's pipeline directly with BBDuk + BBMap defaults)
- Mixed single-ended and pair-ended data, which CIPHER's config format explicitly
  does not handle yet
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: CIPHER
  role: Workflow platform whose default filtering (mapq <20, length <10) and --download_data
    flag define the benchmark conditions
  repo: github.com/c-guzman/cipher-workflow-platform
- name: Trimmomatic
  role: Benchmarked adapter-decontamination/trimming tool compared on retained-read
    yield and runtime
- name: cutadapt
  role: Benchmarked adapter-decontamination/trimming tool compared on retained-read
    yield and runtime
- name: BBDuk
  role: Fastest trimmer in the benchmark; selected by CIPHER as best choice for read
    trimming and adapter removal
- name: BWA
  role: Splice-unaware aligner benchmarked on unique/multimapped/low-quality read
    counts and runtime
- name: Bowtie2
  role: Splice-unaware aligner benchmarked on ENCODE datasets
- name: BBMap
  role: Splice-unaware aligner benchmarked; selected by CIPHER as the default DNA
    mapper
- name: STAR
  role: Splice-aware aligner included for completeness in mapping comparisons
- name: HISAT2
  role: Splice-aware aligner included for completeness in mapping comparisons
- name: FastQC
  role: Raw sequence evaluation / quality control of input FASTQ before trimming
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:b910597ab7e80205b300d624fe5a89f06761c3b0e8a09ee61ae2273c63644037
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# ngs-adapter-trimming-benchmarking

## Summary

A benchmarking method that reproduces CIPHER's evaluation of adapter-decontamination tools (Trimmomatic, cutadapt, BBDuk) and read mappers (BWA, Bowtie2, BBMap, STAR, HISAT2) by recording retained-read yield, unique/multimapped/low-quality read counts, and wall-clock runtime per tool on a shared public FASTQ dataset.

## When to use

Use when choosing between NGS read-trimming or alignment tools (e.g., BBDuk vs cutadapt vs Trimmomatic; BBMap vs BWA vs STAR) and you need speed-vs-accuracy metrics on a common ENCODE FASTQ input.

## When NOT to use

- Input is already trimmed, adapter-free FASTQ where only mapping benchmarking is needed
- The goal is routine production trimming/alignment rather than a tool-selection benchmark (use CIPHER's pipeline directly with BBDuk + BBMap defaults)
- Mixed single-ended and pair-ended data, which CIPHER's config format explicitly does not handle yet

## Inputs

- Public raw FASTQ dataset (e.g., ENCODE ChIP-seq H3K4me1 HCT116 sample)
- Ensembl/Gencode reference genome FASTA and GTF annotation files
- Installed trimming tools: Trimmomatic, cutadapt, BBDuk
- Installed aligners: BWA, Bowtie2, BBMap, STAR, HISAT2

## Outputs

- benchmark_metrics.csv with one metrics row per tool (8 tools): trimming yield, unique/multimapped/low-quality read counts, and wall-clock runtime
- Per-tool retained-read yield after trimming
- Per-aligner unique, multimapped, and low-quality/unmapped read counts

## How to apply

Obtain a public raw FASTQ dataset (e.g., an ENCODE ChIP-seq H3K4me1 HCT116 sample) plus matching Ensembl/Gencode reference FASTA and GTF files, as fetched via CIPHER's --download_data flag. Trim the FASTQ with Trimmomatic, cutadapt, and BBDuk under CIPHER's default filtering (reads with mapq <20 or length <10 removed), recording retained-read yield after each tool. Map the trimmed reads with splice-unaware aligners (BWA, Bowtie2, BBMap) and splice-aware aligners (STAR, HISAT2), parsing each aligner's output statistics for unique, multimapped, and low-quality/unmapped read counts. Record wall-clock runtime per trimming and mapping tool, then compile all per-tool metrics into a single benchmark_metrics.csv. Validation mirrors CIPHER: confirm a metrics row exists for each of the 8 tools and that the best trimmer and mapper reproduce CIPHER's reported selections (BBDuk for trimming/decontamination, BBMap as default aligner).

## Related tools

- **CIPHER** (Workflow platform whose default filtering (mapq <20, length <10) and --download_data flag define the benchmark conditions) — github.com/c-guzman/cipher-workflow-platform
- **Trimmomatic** (Benchmarked adapter-decontamination/trimming tool compared on retained-read yield and runtime)
- **cutadapt** (Benchmarked adapter-decontamination/trimming tool compared on retained-read yield and runtime)
- **BBDuk** (Fastest trimmer in the benchmark; selected by CIPHER as best choice for read trimming and adapter removal)
- **BWA** (Splice-unaware aligner benchmarked on unique/multimapped/low-quality read counts and runtime)
- **Bowtie2** (Splice-unaware aligner benchmarked on ENCODE datasets)
- **BBMap** (Splice-unaware aligner benchmarked; selected by CIPHER as the default DNA mapper)
- **STAR** (Splice-aware aligner included for completeness in mapping comparisons)
- **HISAT2** (Splice-aware aligner included for completeness in mapping comparisons)
- **FastQC** (Raw sequence evaluation / quality control of input FASTQ before trimming)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode preprocessing --config CONFIG --fa ref.fa --gtf genes.gtf --download_data
```

## Evaluation signals

- benchmark_metrics.csv contains a metrics row for each of the 8 tools (3 trimmers + 5 aligners)
- BBDuk emerges as fastest trimmer with high retained-read yield, reproducing CIPHER's selection of BBDuk for trimming and adapter removal
- BBMap reproduces as the best-performing default aligner among BWA, Bowtie2, and BBMap on ENCODE data
- Every trimming run reports a retained-read yield; every aligner reports unique, multimapped, and low-quality/unmapped read counts plus runtime
- All tools were run on the same raw FASTQ input and same reference FASTA/GTF, so per-tool metrics are directly comparable

## Limitations

- Software version numbers for the tested tools are not reported; all mappers were run in Teaser's default mode with no additional parameters unless otherwise indicated
- Results are tied to the chosen genome build (UCSC hg19/hg38) and ENCODE ChIP-seq input; other read types may change tool rankings
- Runtime comparisons are environment-dependent (CIPHER processed six ~30M-read ChIP-seq datasets in 7 h 23 min on its benchmark machine)

## Evidence

- [full_text] BBDuk was the fastest trimmer and, combined with its accuracy, ease of use, and support for single- and pair-ended data, was selected by CIPHER as the best choice for read trimming and adapter removal.: [evidence span withheld — anchor status: not_found]
- [methods] providing the “--download_data” flag will automatically download the appropriate Ensembl/Gencode reference files: "providing the “--download_data” flag will automatically download the appropriate Ensembl/Gencode reference files"
- [abstract] raw sequence evaluation, read trimming and adapter removal, read mapping and quality filtering, visualization track generation, and extensive quality control assessment: "(2) read trimming and adapter removal, (3) read mapping and quality filtering"
- [readme] DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES NOT HANDLE THIS USE-CASE YET.: [evidence span withheld — anchor status: external_doc]
- [results] All mappers were run in Teaser’s default mode with no additional parameters unless otherwise indicated.: "All mappers were run in Teaser’s default mode with no additional parameters unless otherwise indicated."
