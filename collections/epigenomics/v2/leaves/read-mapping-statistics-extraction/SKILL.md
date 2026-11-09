---
name: read-mapping-statistics-extraction
description: 'Decision support for choosing a DNA/RNA read aligner: when you need
  to compare mappers on the same FASTQ input by unique vs multimapped vs low-quality/unmapped
  read counts and runtime.'
when_to_use_negative:
- You only need a single mapping result for downstream peak calling — use one aligner
  directly (CIPHER defaults to BBMap) rather than a comparative benchmark.
- The question is adapter/trimming performance (Trimmomatic vs cutadapt vs BBDuk),
  which is a separate trimming-yield comparison.
- Input is not FASTQ sequence reads against a reference genome (e.g. already-assembled
  or count-matrix data).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: BBMap
  role: Splice-unaware DNA mapper benchmarked; CIPHER's default aligner
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: BWA
  role: Splice-unaware DNA mapper compared in the benchmark
- name: Bowtie2
  role: Splice-unaware DNA mapper compared; used to map reads to UCSC hg19
- name: STAR
  role: Splice-aware aligner included for completeness
- name: HISAT2
  role: Splice-aware aligner included for completeness
- name: CIPHER
  role: Workflow platform whose read-mapping and quality-filtering step (mapq <20,
    length <10) supplies the thresholds and automation
  repo: https://github.com/c-guzman/cipher-workflow-platform
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:e502d6e22fe43cf643e3c2ff335cbca048374570cef4f7d27945f584e8dd2cba
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Read-mapping statistics extraction

## Summary

Parses the output of NGS read aligners (BBMap, BWA, Bowtie2 for splice-unaware; STAR, HISAT2 for splice-aware) to tabulate per-tool counts of unique, multimapped, and low-quality/unmapped reads, enabling head-to-head mapper benchmarking as performed in the CIPHER platform.

## When to use

Decision support for choosing a DNA/RNA read aligner: when you need to compare mappers on the same FASTQ input by unique vs multimapped vs low-quality/unmapped read counts and runtime.

## When NOT to use

- You only need a single mapping result for downstream peak calling — use one aligner directly (CIPHER defaults to BBMap) rather than a comparative benchmark.
- The question is adapter/trimming performance (Trimmomatic vs cutadapt vs BBDuk), which is a separate trimming-yield comparison.
- Input is not FASTQ sequence reads against a reference genome (e.g. already-assembled or count-matrix data).

## Inputs

- Trimmed FASTQ files (single- or pair-ended, e.g. ENCODE ChIP-seq H3K4me1 HCT116 reads)
- Reference genome FASTA (e.g. UCSC hg19 or hg38)
- Per-aligner mapping output/statistics logs (BWA, Bowtie2, BBMap, STAR, HISAT2)

## Outputs

- Per-tool counts of unique, multimapped, and low-quality/unmapped reads
- Wall-clock runtime per mapper
- Consolidated metrics table (benchmark_metrics.csv) with one row per aligner

## How to apply

Map the same trimmed FASTQ reads to a single reference genome (e.g. UCSC hg38, as CIPHER used for simulated reads; UCSC hg19 for ENCODE ChIP-seq) with each aligner under fixed settings — in CIPHER's benchmark all mappers ran in Teaser's default mode with no additional parameters. Run the splice-unaware aligners BWA, Bowtie2, and BBMap and, for completeness, the splice-aware aligners STAR and HISAT2 on identical input. Parse each aligner's output statistics to extract the number of unique, multimapped, and low-quality/unmapped reads per tool, plus wall-clock runtime. CIPHER's default filtering removes low-quality (mapq <20) and short (length <10) reads, so apply the same threshold consistently so counts are comparable across mappers. Compile all per-tool metrics into a single table (benchmark_metrics.csv) and confirm a metrics row exists for each tool tested.

## Related tools

- **BBMap** (Splice-unaware DNA mapper benchmarked; CIPHER's default aligner) — https://github.com/c-guzman/cipher-workflow-platform
- **BWA** (Splice-unaware DNA mapper compared in the benchmark)
- **Bowtie2** (Splice-unaware DNA mapper compared; used to map reads to UCSC hg19)
- **STAR** (Splice-aware aligner included for completeness)
- **HISAT2** (Splice-aware aligner included for completeness)
- **CIPHER** (Workflow platform whose read-mapping and quality-filtering step (mapq <20, length <10) supplies the thresholds and automation) — https://github.com/c-guzman/cipher-workflow-platform

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH>
```

## Evaluation signals

- benchmark_metrics.csv contains a metrics row for each mapper tested (BWA, Bowtie2, BBMap, STAR, HISAT2).
- Unique + multimapped + low-quality/unmapped read counts sum consistently to the trimmed input read count per mapper.
- All mappers were run on the same trimmed FASTQ and reference (e.g. UCSC hg38) under identical default parameters, so counts are directly comparable.
- The quality-filter cutoff (mapq <20, length <10) was applied uniformly across tools before counting retained reads.
- The best-performing mapper reproduces CIPHER's reported default aligner selection (BBMap).

## Limitations

- CIPHER benchmark mappers were run in Teaser's default mode with no additional parameters unless otherwise indicated, and software version numbers for the tested tools are not reported in the article.
- Benchmark comparisons depend on the reference build (hg19 vs hg38) and dataset type; DNA-mapper results (BBMap, BWA, Bowtie2 on ENCODE data) do not directly generalize to splice-aware RNA-seq mapping.
- CIPHER currently only runs entire workflows, not subsets of tools, so statistics extraction as a standalone comparison requires running the pipelines outside CIPHER or post-processing its QC outputs.

## Evidence

- [results] DNA mappers (BBMap, BWA, and Bowtie2) using ENCODE datasets: "DNA mappers (BBMap, BWA, and Bowtie2) using ENCODE datasets"
- [methods] Mapping or alignment, while generally being the most computationally intensive part of any pipeline: "Mapping or alignment, while generally being the most computationally intensive part of any pipeline"
- [results] The simulated reads were then mapped to the entire UCSC hg38 reference genome: "The simulated reads were then mapped to the entire UCSC hg38 reference genome"
- [results] All mappers were run in Teaser's default mode with no additional parameters unless otherwise indicated.: "All mappers were run in Teaser’s default mode with no additional parameters unless otherwise indicated."
- [abstract] read mapping and quality filtering: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
