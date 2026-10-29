---
name: fastq-read-alignment-quality-assessment
description: Use when starting from raw ChIP-seq or DNase-seq FASTQ (treatment and
  control) and you need aligned BAMs plus early QC metrics (read quality, GC content,
  library complexity PBC, fragment size) before peak calling.
when_to_use_negative:
- Input is already aligned BAM data and only downstream peak calling or annotation
  (steps 5-9) is needed
- The goal is replicate consistency assessment alone (wiggle correlation / peak overlap)
  rather than the read-level alignment QC core
- Peak number is under 200 and only MDSeqPos motif analysis behavior is of interest
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChiLin
  role: Unified pipeline that orchestrates FastQC, mapping, fragment-size estimation
    and PBC evaluation as steps 1-4
  repo: https://github.com/cfce/chilin
- name: BWA
  role: Default mapper in step 2; mapping cannot be skipped because it produces the
    necessary BAM files
  repo: https://github.com/lh3/bwa
- name: FastQC
  role: Step 1 sequence quality evaluation, reads GC contents evaluation and library
    contamination check
- name: MACS2
  role: 'Step 3: subsample BAM files and do MACS2 fragment size estimation'
  repo: https://github.com/taoliu/MACS/
- name: samtools
  role: BAM manipulation dependency installed via conda for the ChiLin environment
- name: seqtk
  role: FASTQ/FASTA processing dependency of the ChiLin environment
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:c3ebe49cf3b5cafeabd7d006a960044e5c9fa89016c2b582c6271639e30b649b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - BWA
  - ChiLin
  - FastQC
  - MACS2
  - bedtools
  - samtools
  - seqtk
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fastq-read-alignment-quality-assessment

## Summary

Runs the mandatory ChiLin QC core on ChIP-seq/DNase-seq FASTQ inputs: FastQC sequence-quality evaluation, BWA (or bowtie/star) mapping, MACS2 fragment-size estimation on 4M-read-subsampled BAMs, and PBC evaluation, producing the BAM and QC metrics that feed downstream replicate-consistency and FRiP reporting.

## When to use

Use when starting from raw ChIP-seq or DNase-seq FASTQ (treatment and control) and you need aligned BAMs plus early QC metrics (read quality, GC content, library complexity PBC, fragment size) before peak calling.

## When NOT to use

- Input is already aligned BAM data and only downstream peak calling or annotation (steps 5-9) is needed
- The goal is replicate consistency assessment alone (wiggle correlation / peak overlap) rather than the read-level alignment QC core
- Peak number is under 200 and only MDSeqPos motif analysis behavior is of interest

## Inputs

- Treatment replicate FASTQ files (e.g. treat1.fastq, treat2.fastq)
- Control replicate FASTQ files (e.g. control1.fastq, control2.fastq)
- Species-specific dependent data archive (hg19/hg38/mm9/mm10 .tgz) extracted into db/

## Outputs

- Aligned BAM files for treatment and control samples
- FastQC sequence-quality, GC-content and library-contamination evaluation
- MACS2 fragment-size estimate on subsampled BAMs
- PBC (PCR bottlenecking coefficient) evaluation metrics
- id_summary.txt QC statistics and id_report.pdf QC report

## How to apply

Install ChiLin (git clone http://github.com/cfce/chilin/ and python setup.py install) after provisioning bwa=0.7.13, samtools=0.1.19, bedtools=2.17.0, fastqc, macs2=2.1.0 via conda, and download species-specific dependent data (e.g. wget -c http://cistrome.org/chilin/_downloads/hg19.tgz) into db/, verifying with md5sum -c. Launch in simple mode supplying treatment and control FASTQ replicates, peak type (narrow/broad) and factor class; step 1 FastQC evaluates sequence quality, GC contents and library contamination and is skippable, while step 2 mapping (bwa default, bowtie or star) cannot be skipped because it produces the necessary BAM files. Steps 3-4 subsample BAMs for MACS2 fragment-size estimation and PBC evaluation (step 4 is skipped if step 3 ran). Accept the run when BAMs are produced and the resulting QC statistics in id_summary.txt are consistent with the paper's reference ranges for the corresponding case-study dataset.

## Related tools

- **ChiLin** (Unified pipeline that orchestrates FastQC, mapping, fragment-size estimation and PBC evaluation as steps 1-4) — https://github.com/cfce/chilin
- **BWA** (Default mapper in step 2; mapping cannot be skipped because it produces the necessary BAM files) — https://github.com/lh3/bwa
- **FastQC** (Step 1 sequence quality evaluation, reads GC contents evaluation and library contamination check)
- **MACS2** (Step 3: subsample BAM files and do MACS2 fragment size estimation) — https://github.com/taoliu/MACS/
- **samtools** (BAM manipulation dependency installed via conda for the ChiLin environment)
- **seqtk** (FASTQ/FASTA processing dependency of the ChiLin environment) — https://github.com/lh3/seqtk

## Examples

```
chilin simple -u user -s species --threads 8 -i id -o output -t treat1.fastq,treat2.fastq -c control1.fastq,control2.fastq -p narrow -r tf
```

## Evaluation signals

- Step 2 mapping completes and produces BAM files for every input FASTQ replicate (mapping cannot be skipped)
- PBC evaluation output exists and step 4 is correctly auto-skipped when step 3 fragment-size estimation already ran
- id_summary.txt is generated with FRiP scores, fragment sizes and PBC values computable for each sample
- FastQC report shows no severe read-quality failure, GC-content bias, or library contamination for the submitted FASTQs
- Computed QC statistics fall within the paper's reported reference ranges (e.g. H3K27me3 FRiP 0.04-0.32%) when reproducing case-study datasets

## Limitations

- Only steps 1 and 2 (mapping) plus merged peak calling (step 6) are mandatory; skippable steps mean reports may lack some QC layers if not explicitly enabled
- No software version is stated for ChiLin itself in the paper's discussion section
- Reproducibility is claimed 'with only a simple command' but the discussion section provides no concrete commands, environment specification, or output artifacts
- Steps 3-4 operate on subsampled BAM files, so fragment-size and PBC metrics are estimates rather than full-data values

## Evidence

- [methods] FastQC sequence quality, GC content and library contamination: "step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination"
- [methods] mapping cannot be skipped: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [methods] fragment size estimation and PBC: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [methods] PBC evaluation on subsampled BAMs: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [readme] conda environment provisioning: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0 bioconductor-seqlogo"
- [methods] dependent data download: "wget -c http://cistrome.org/chilin/_downloads/hg19.tgz"
