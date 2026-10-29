---
name: unique-mapping-ratio-qc-statistics
description: Use when screening newly produced ChIP-seq or DNase-seq FASTQ/BAM files
  for mapping quality, or when comparing a new sample's uniquely mapped read ratio
  against cumulative historical QC distributions to decide if it passes the >0.5 cutoff.
when_to_use_negative:
- The goal is peak-level QC such as FRiP scores, PBC, or replicate overlap — use the
  corresponding ChiLin steps instead
- The sample is not a short-read ChIP-seq/DNase-seq library aligned to a supported
  reference (hg19/hg38/mm9/mm10), since the cutoff was derived from the public ChIP-seq/DNase-seq
  quality atlas
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: ChiLin
  role: Pipeline that orchestrates the bwa mapping step (step 2) and compiles the
    per-sample mapping QC records
  repo: https://github.com/cfce/chilin
- name: bwa
  role: Default aligner used in step 2 to produce BAM files and uniquely mapped read
    statistics
  repo: https://github.com/lh3/bwa
- name: samtools
  role: SAM/BAM manipulation supporting mapping-statistics evaluation within the pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:8b18e1b0c075481771b2948602a632640e98fb3f62a9f0278e98f6429aeb71ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - R
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# unique-mapping-ratio-qc-statistics

## Summary

Computes and interprets the uniquely mapped read ratio — the fraction of sequencing reads aligning uniquely to the reference genome via bwa within the ChiLin pipeline — as a read-layer ChIP-seq/DNase-seq QC metric, benchmarked against a historical atlas of 23,677 public samples.

## When to use

Use when screening newly produced ChIP-seq or DNase-seq FASTQ/BAM files for mapping quality, or when comparing a new sample's uniquely mapped read ratio against cumulative historical QC distributions to decide if it passes the >0.5 cutoff.

## When NOT to use

- The goal is peak-level QC such as FRiP scores, PBC, or replicate overlap — use the corresponding ChiLin steps instead
- The sample is not a short-read ChIP-seq/DNase-seq library aligned to a supported reference (hg19/hg38/mm9/mm10), since the cutoff was derived from the public ChIP-seq/DNase-seq quality atlas

## Inputs

- FASTQ read files (single- or paired-end) from ChIP-seq or DNase-seq samples
- Reference genome index for bwa (hg19, hg38, mm9, or mm10)
- Per-sample mapping statistics JSON records (id_map.) from ChiLin step 2

## Outputs

- BAM files of aligned reads per sample
- Per-sample uniquely mapped read ratio statistics
- Cumulative distribution and category-level cutoff table (e.g. qc_metric_summary.csv) showing fraction of samples with unique ratio > 0.5

## How to apply

Run ChiLin step 2 mapping with bwa (default; bowtie or star also supported) on each sample's FASTQ files to produce a BAM and per-sample mapping statistics (id_map. records). Parse the mapping-ratio statistics (JSON) for each sample and compute the uniquely mapped read ratio = uniquely mapped reads / total reads. Interpret the ratio against the compiled quality atlas: more than 85% of the 23,677 public samples across eight literature-based categories have a uniquely mapped ratio above 0.5, so 0.5 serves as a usable assay-type-insensitive QC cutoff for ChIP-seq data. Compare new samples against the historical distribution and flag any category whose cumulative fraction deviates from the reported thresholds.

## Related tools

- **ChiLin** (Pipeline that orchestrates the bwa mapping step (step 2) and compiles the per-sample mapping QC records) — https://github.com/cfce/chilin
- **bwa** (Default aligner used in step 2 to produce BAM files and uniquely mapped read statistics) — https://github.com/lh3/bwa
- **samtools** (SAM/BAM manipulation supporting mapping-statistics evaluation within the pipeline)

## Examples

```
conda create -n chilin_env python=2; conda install bwa=0.7.13 samtools=0.1.19 fastqc macs2=2.1.0; chilin -h
```

## Evaluation signals

- More than 85% of samples in each category should show a uniquely mapped read ratio above 0.5, matching the reported cumulative cutoff
- Every sample has a parsed id_map. record with a defined unique-mapped ratio and non-zero total reads
- Cumulative fraction per category reproduces the expected distribution from the 23,677-sample atlas; flag categories that deviate
- Mapping BAM files exist for all samples (step 2 cannot be skipped)

## Limitations

- The >0.5 cutoff is an empirical cumulative threshold from historical data, not a hard pass/fail boundary; individual low-quality categories may deviate
- ChiLin version is not stated, and reproducibility commands/environment specs are not provided in the discussion section
- The metric is read-layer only — a passing mapping ratio does not guarantee downstream QC success (e.g. FRiP, PBC, replicate concordance)

## Evidence

- [abstract] Unique ratio cutoff derivation: [evidence span withheld — anchor status: not_found]
- [methods] Mapping step description: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [abstract] Sample corpus size: "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [readme] Conda installation of bwa for the environment: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
