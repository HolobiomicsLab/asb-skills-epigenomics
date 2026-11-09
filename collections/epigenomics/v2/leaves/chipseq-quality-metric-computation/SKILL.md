---
name: chipseq-quality-metric-computation
description: Use when processing FASTQ/BAM ChIP-seq or DNase-seq samples through ChiLin-style
  QC steps and you need per-sample read-layer metrics (FastQC score, uniquely mapped
  ratio, PBC, FRiP, fragment size) for quality assessment.
when_to_use_negative:
- Input data is not ChIP-seq or DNase-seq (e.g., RNA-seq or long-read data) — the
  pipeline's QC metrics and historical atlas cover only ChIP-seq and DNase-seq samples
- Mapping results already exist in another format and bwa cannot be run — step 2 cannot
  be skipped because it provides the necessary BAM files
- The goal is motif analysis on a small peak set — MDSeqPos motif analysis is automatically
  skipped for peak numbers less than 200, so ChiLin QC steps alone are insufficient
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ChiLin
  role: Unified command-line pipeline that automates the QC steps (FastQC, mapping,
    PBC, FRiP) and report generation
  repo: https://github.com/cfce/chilin
- name: FastQC
  role: 'Step 1: sequence quality evaluation, GC contents evaluation and library contamination
    check'
- name: bwa
  role: 'Step 2: default mapper producing the BAM files required by all later QC steps'
  repo: https://github.com/lh3/bwa
- name: MACS2
  role: 'Step 3: fragment size estimation on sub-sampled BAM files; also the default
    peak caller'
  repo: https://github.com/taoliu/MACS
- name: samtools
  role: BAM manipulation in the recommended conda environment for the pipeline
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:01c7ca31cdf97177298f8173f400a8b190ace40c580cf3cfc7c13d4217975361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - FastQC
  - MACS2
  - R
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# chipseq-quality-metric-computation

## Summary

Compute the layered ChiLin quality-control metrics for ChIP-seq and DNase-seq experiments (FastQC sequence quality, bwa mapping ratio, macs2 fragment-size estimation, PBC, replicates overlap/correlation, merged peak calling, and FRiP scores), benchmarking new samples against the compiled 23,677-sample quality atlas.

## When to use

Use when processing FASTQ/BAM ChIP-seq or DNase-seq samples through ChiLin-style QC steps and you need per-sample read-layer metrics (FastQC score, uniquely mapped ratio, PBC, FRiP, fragment size) for quality assessment.

## When NOT to use

- Input data is not ChIP-seq or DNase-seq (e.g., RNA-seq or long-read data) — the pipeline's QC metrics and historical atlas cover only ChIP-seq and DNase-seq samples
- Mapping results already exist in another format and bwa cannot be run — step 2 cannot be skipped because it provides the necessary BAM files
- The goal is motif analysis on a small peak set — MDSeqPos motif analysis is automatically skipped for peak numbers less than 200, so ChiLin QC steps alone are insufficient

## Inputs

- ChIP-seq or DNase-seq FASTQ read files
- BAM alignment files produced by bwa (default) / bowtie / star mapping
- Reference genome data packages (hg19, hg38, mm9, mm10 tgz downloads from cistrome.org)
- Sub-sampled BAM files for macs2 fragment-size estimation and PBC evaluation
- Replicate peak call files for overlap/correlation analysis

## Outputs

- Per-sample QC JSON records (id_fastqc., id_map., id_pbc., id_frag., id_rep., id_frip.)
- QC report with comparisons against historical reference metrics
- Fragment-size estimate from macs2
- PBC (PBC score) per sample
- FRiP score per sample
- Merged peak calls from step 6 (input to bedAnnotate.py meta-region distribution, step 8)

## How to apply

Run the ChiLin pipeline in order: step 1 FastQC sequence quality evaluation (optional, skippable), step 2 bwa mapping (default; cannot be skipped because it produces the required BAM files), step 3 sub-sample BAM files and do macs2 fragment size estimation, step 4 PBC evaluation on the sub-sampled BAMs (skipped if step 3 ran), step 5 replicates peak overlap/correlation analysis, step 6 merged peak calling (cannot be skipped as it feeds annotation), and step 7 FRiP score calculation per sample. Each step emits per-sample QC JSON records with id_fastqc., id_map., id_pbc., id_frag., id_rep., and id_frip. prefixes. Interpret new samples against the historical quality atlas of 23,677 public samples (11,265 datasets) across eight literature-based categories, where usable cutoffs are quality score >25, uniquely mapped read ratio >0.5, and PBC >0.8, with median read depths of 25M for ChIP-seq and 40M for DNase-seq. The rationale is that read-layer metrics are largely assay-type independent, so a sample far below these cutoffs likely has a genuine library or sequencing problem worth investigating before downstream analysis.

## Related tools

- **ChiLin** (Unified command-line pipeline that automates the QC steps (FastQC, mapping, PBC, FRiP) and report generation) — https://github.com/cfce/chilin
- **FastQC** (Step 1: sequence quality evaluation, GC contents evaluation and library contamination check)
- **bwa** (Step 2: default mapper producing the BAM files required by all later QC steps) — https://github.com/lh3/bwa
- **MACS2** (Step 3: fragment size estimation on sub-sampled BAM files; also the default peak caller) — https://github.com/taoliu/MACS
- **samtools** (BAM manipulation in the recommended conda environment for the pipeline)

## Examples

```
git clone https://github.com/cfce/chilin; cd chilin; docker pull qq06/chilin:1.0; bash demo/docker_test.sh
```

## Evaluation signals

- All six per-sample QC JSON files exist (id_fastqc., id_map., id_pbc., id_frag., id_rep., id_frip.) after a full run
- Cumulative QC distributions reproduce the atlas cutoffs: >85% of samples with sequence quality score >25, uniquely mapped ratio >0.5, and PBC >0.8
- Median read depth of the reference set matches 25M for ChIP-seq and 40M for DNase-seq
- New samples can be compared against the historical QC distribution via the id_bwa_compare R script (id_bwa_compare.R -> id_bwa_compare.pdf)
- FRiP–peak-number correlation is high for accessibility/TF/chromatin-regulator samples, ~0.37 for histone modification samples, and negative for DNase FRiP vs estimated fragment size

## Limitations

- No software version is stated for ChiLin in the article, complicating exact reproduction
- FastQC evaluation (step 1) and several downstream analyses (PhastCons distribution, regulatory potential, MDSeqPos motif analysis for peaks <200) are optional/skippable, so a 'complete' run is ambiguous
- MDSeqPos motif analysis is automatically skipped when peak numbers are fewer than 200

## Evidence

- [abstract] ChiLin automates QC in unified framework: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [abstract] Atlas scope: "We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users."
- [methods] FastQC step: "step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination"
- [methods] PBC step: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [methods] FRiP step: "step 7: calculate FRiP scores for each sample."
- [readme] Environment versions: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
