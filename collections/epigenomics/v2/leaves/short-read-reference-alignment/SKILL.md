---
name: short-read-reference-alignment
description: Use when raw or adapter-trimmed FASTQ reads from ChIP-seq, RNA-seq, DNase-seq,
  MNase-seq, ATAC-seq, or GRO-seq datasets must be aligned to a reference genome to
  produce sorted BAMs with alignment statistics.
when_to_use_negative:
- Input is already a sorted BAM or otherwise aligned — the mapping step would be redundant;
  proceed to mode-specific downstream analysis instead.
- De novo transcriptome assembly is the goal — CIPHER explicitly has no pipeline for
  de novo transcriptome assembly (planned future work).
- Mixed single- and pair-ended data in one run — CIPHER does not handle mixing single-
  and pair-ended data in the same CONFIG file.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: BBMap
  role: Default short-read aligner in CIPHER, selected as best performer after benchmarking
    against BWA, Bowtie2, STAR, and HISAT2
  repo: c-guzman/cipher-workflow-platform
- name: BWA
  role: Splice-unaware alternative aligner selectable via the --aligner flag
- name: Bowtie2
  role: Splice-unaware alternative aligner selectable via the --aligner flag
- name: STAR
  role: Splice-aware alternative aligner for RNA-seq-type data selectable via the
    --aligner flag
- name: HISAT2
  role: Splice-aware alternative aligner selectable via the --aligner flag
- name: BBDuk
  role: Upstream trimming/filtering tool whose retained reads (mapq < 20, length <
    10 cutoffs) feed the alignment step
  repo: c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow engine orchestrating the mapping stage inside CIPHER's pre-processing
    pipeline
  repo: c-guzman/cipher-workflow-platform
- name: Docker
  role: Containerization technology used to run the alignment tools reproducibly (README
    documents Singularity as the container runtime)
- name: MultiQC
  role: Aggregates alignment and QC statistics into pdf/html reports in the output
    directory
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:68f8ea24f822bff63cfa340910350b9888de905c58dc732684d464f56bc6df9f
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
  - Docker
  - FastQC
  - HISAT2
  - MACS2
  - MultiQC
  - Nextflow
  - STAR
  - Trimmomatic
  - cutadapt
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 21
---

# short-read-reference-alignment

## Summary

Maps trimmed NGS reads (single- or pair-ended FASTQ) to a reference genome as stage 3 of the CIPHER pre-processing pipeline, using BBMap as the default aligner with BWA, Bowtie2, STAR, and HISAT2 as selectable alternatives.

## When to use

Use when raw or adapter-trimmed FASTQ reads from ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq, or GRO-seq datasets must be aligned to a reference genome to produce sorted BAMs with alignment statistics.

## When NOT to use

- Input is already a sorted BAM or otherwise aligned — the mapping step would be redundant; proceed to mode-specific downstream analysis instead.
- De novo transcriptome assembly is the goal — CIPHER explicitly has no pipeline for de novo transcriptome assembly (planned future work).
- Mixed single- and pair-ended data in one run — CIPHER does not handle mixing single- and pair-ended data in the same CONFIG file.

## Inputs

- Adapter-trimmed FASTQ files (single-ended or pair-ended, gzipped), per sample/replicate from the CONFIG file
- Reference genome FASTA (e.g. UCSC hg19/hg38 or Ensembl/Gencode, fetchable via the --download_data flag)
- CONFIG tab-separated file specifying MergeID, SampleID, FASTQ path(s), InputID, and Mark columns
- Aligner choice via --aligner (default BBMap; alternatives bwa, bowtie2, star, hisat2)

## Outputs

- Sorted BAM files per sample/replicate
- Alignment statistics (unique, multimapped, low-quality/unmapped read counts)
- Artifacts emitted to the './report' output directory (override with --outdir), aggregated by MultiQC QC reports

## How to apply

After raw sequence evaluation (FastQC) and read trimming/adapter removal (BBDuk, which filters low-quality reads with mapq < 20 and short reads with length < 10), map the retained reads to the reference genome (e.g. UCSC hg19/hg38 or Ensembl/Gencode FASTA provided via the --fasta/--fa flag). CIPHER's default aligner is BBMap, chosen after benchmarking against BWA, Bowtie2 (splice-unaware) and STAR, HISAT2 (splice-aware) for speed and accuracy; alternatives are selected via the --aligner flag. Run the alignment inside the Nextflow workflow with Docker or Singularity containers so the step is reproducible. Collect the resulting sorted BAM files together with each aligner's statistics on unique, multimapped, and low-quality/unmapped reads, and propagate them to quality control aggregation (MultiQC) and downstream mode-specific analysis (e.g. MACS2 peak calling for ChIP-seq).

## Related tools

- **BBMap** (Default short-read aligner in CIPHER, selected as best performer after benchmarking against BWA, Bowtie2, STAR, and HISAT2) — c-guzman/cipher-workflow-platform
- **BWA** (Splice-unaware alternative aligner selectable via the --aligner flag)
- **Bowtie2** (Splice-unaware alternative aligner selectable via the --aligner flag)
- **STAR** (Splice-aware alternative aligner for RNA-seq-type data selectable via the --aligner flag)
- **HISAT2** (Splice-aware alternative aligner selectable via the --aligner flag)
- **BBDuk** (Upstream trimming/filtering tool whose retained reads (mapq < 20, length < 10 cutoffs) feed the alignment step) — c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow engine orchestrating the mapping stage inside CIPHER's pre-processing pipeline) — c-guzman/cipher-workflow-platform
- **Docker** (Containerization technology used to run the alignment tools reproducibly (README documents Singularity as the container runtime))
- **MultiQC** (Aggregates alignment and QC statistics into pdf/html reports in the output directory)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.txt --fa ref.fa --gtf ref.gtf --lib s --readLen 75
```

## Evaluation signals

- Sorted BAM files with alignment statistics are present in the './report' output directory after the run completes.
- Per-tool metrics (unique, multimapped, low-quality/unmapped read counts and runtime) can be tabulated per aligner, e.g. as benchmark_metrics.csv.
- Benchmarking reproduces CIPHER's reported selection: BBMap as the default mapper against ENCODE datasets.
- QC reports aggregated by MultiQC (pdf or html) include the alignment step's statistics.
- The workflow completes without abrupt termination — on a local machine, memory pressure must be controlled via the -qs <INT> flag.

## Limitations

- Mapping is the most computationally intensive part of the pipeline; running locally without limiting parallelization (-qs) can exhaust memory and abruptly end the workflow.
- Aligner benchmarking was performed with Teaser's default mode and simulated reads mapped to the entire UCSC hg38 reference genome, so version numbers for tools tested were not reported.
- Benchmarking used ENCODE datasets (BBMap, BWA, Bowtie2) mapped to UCSC hg19 via Bowtie; other genomes may behave differently.

## Evidence

- [abstract] read mapping and quality filtering: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [results] benchmarking of mappers: "DNA mappers (BBMap, BWA, and Bowtie2) using ENCODE datasets"
- [results] hg38 mapping of simulated reads: "The simulated reads were then mapped to the entire UCSC hg38 reference genome"
- [methods] mapping is computationally intensive: "Mapping or alignment, while generally being the most computationally intensive part of any pipeline"
- [readme] execution command: [evidence span withheld — anchor status: external_doc]
