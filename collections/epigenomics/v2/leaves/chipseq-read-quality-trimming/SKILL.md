---
name: chipseq-read-quality-trimming
description: Use when starting from raw ChIP-seq FASTQ files (e.g. ENA/ENCODE IP and
  control libraries) that must be cleaned prior to alignment with a short-read aligner
  such as Bowtie, especially when computing read coverage or IP/control enrichment
  over genomic intervals like HOT regions.
when_to_use_negative:
- Input is already trimmed FASTQ or an alignment file (BAM/SAM) — trimming is redundant
  and the pipeline should continue at the alignment or coverage stage
- The goal is quantifying signal on ordinary (non-HOT) peak calls without artifact
  concerns — the trimming rationale here is tied to the false-positive/HOT-region
  enrichment analysis
- Reads are single-cell or long-read data — bbduk short-read adapter trimming as used
  in this workflow is not the appropriate choice
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: bbduk
  role: Trims adapters and low-quality bases from ChIP-seq reads before alignment
- name: Bowtie
  role: Aligns the trimmed reads to the reference genome
- name: samtools
  role: Filters, sorts and indexes the resulting alignments, removing unmapped and
    multimapping reads
- name: Bedtools
  role: Computes read coverage over the HOT region intervals from the filtered alignments
- name: genomation
  role: R/Bioconductor package used to compute log2(IP/control) fold enrichment over
    HOT regions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:dfbead1dda0d9b645a0c5bf263cf852ab6a03c7c86e21c43e1c69aa7d2e7ca89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - Bowtie
  - bbduk
  - genomation
  - samtools
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# chipseq-read-quality-trimming

## Summary

Preprocess ChIP-seq FASTQ reads (IP and matching control) by trimming adapters and low-quality bases with bbduk before genome alignment, so that downstream IP/control enrichment measurements (e.g. log2 fold enrichment over HOT regions) are not inflated by adapter contamination or poor-quality bases.

## When to use

Use when starting from raw ChIP-seq FASTQ files (e.g. ENA/ENCODE IP and control libraries) that must be cleaned prior to alignment with a short-read aligner such as Bowtie, especially when computing read coverage or IP/control enrichment over genomic intervals like HOT regions.

## When NOT to use

- Input is already trimmed FASTQ or an alignment file (BAM/SAM) — trimming is redundant and the pipeline should continue at the alignment or coverage stage
- The goal is quantifying signal on ordinary (non-HOT) peak calls without artifact concerns — the trimming rationale here is tied to the false-positive/HOT-region enrichment analysis
- Reads are single-cell or long-read data — bbduk short-read adapter trimming as used in this workflow is not the appropriate choice

## Inputs

- Raw ChIP-seq FASTQ files for IP and matching control libraries (e.g. downloaded from ENA/ENCODE)
- Adapter sequence reference (e.g. Illumina adapter set used by bbduk)
- Reference genome index for downstream Bowtie alignment

## Outputs

- Trimmed FASTQ files for IP and control libraries
- Downstream: BAM files filtered/sorted/indexed with samtools (unmapped and multimapping reads removed)
- Per-experiment log2(IP/control) enrichment values over HOT regions (e.g. ko_hot_enrichment.csv) with enrichment labels

## How to apply

Download the IP and matching control ChIP-seq FASTQ files (e.g. from ENA using accession numbers in Supplementary Table S1) and run bbduk to trim adapters and low-quality bases from the reads. The trimmed reads are then aligned to the reference genome with Bowtie, and the resulting alignments are filtered, sorted and indexed with samtools (removing unmapped and multimapping reads) so that only high-confidence uniquely mapped reads feed into coverage computation. This trimming step matters because downstream quantification — e.g. Bedtools read coverage over HOT region intervals and genomation-computed log2(IP/control) fold enrichment — assumes that signal derives from genuine fragments rather than adapter read-through or low-quality base calls. Per-experiment enrichment values over the target intervals should be written to a table (e.g. ko_hot_enrichment.csv) and interpreted against the known artifact risk that KO ChIP-seq data still show enrichment on HOT regions. Treat a lack of signal in a KO experiment cautiously: it can also reflect technical conditions such as low read number, low library complexity or unsuccessful IP rather than true absence of artifact.

## Related tools

- **bbduk** (Trims adapters and low-quality bases from ChIP-seq reads before alignment)
- **Bowtie** (Aligns the trimmed reads to the reference genome)
- **samtools** (Filters, sorts and indexes the resulting alignments, removing unmapped and multimapping reads)
- **Bedtools** (Computes read coverage over the HOT region intervals from the filtered alignments)
- **genomation** (R/Bioconductor package used to compute log2(IP/control) fold enrichment over HOT regions)

## Examples

```
bbduk.sh in=ko_IP_R1.fq.gz in2=ko_IP_R2.fq.gz out=trimmed_R1.fq.gz out2=trimmed_R2.fq.gz ref=adapters ktrim=r k=23 mink=11 hdist=1 tpe tbo
```

## Evaluation signals

- Trimmed reads contain no residual adapter sequence and low-quality ends are removed before Bowtie alignment
- After samtools filtering, alignments contain no unmapped or multimapping reads; files are sorted and indexed
- Per-experiment log2(IP/control) values over HOT regions are produced for all experiments (e.g. 25 KO ChIP-seq experiments) and written to ko_hot_enrichment.csv
- Validation expectation from the source study: more than half of the 25 KO ChIP-seq experiments show clear enrichment over HOT regions, consistent with reported false positive signals
- Experiments lacking enrichment should be checked for technical causes (low number of reads, low library complexity, unsuccessful IP) before concluding true absence of signal

## Limitations

- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes
- A lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP
- Even after careful trimming and alignment, KO ChIP-seq data still show false positive enrichment on HOT regions, so trimming alone cannot remove this artifact

## Evidence

- [abstract] Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions.: "Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions."
- [discussion] lack of signal may be technical: "a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP."
- [readme] repository scripts: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
