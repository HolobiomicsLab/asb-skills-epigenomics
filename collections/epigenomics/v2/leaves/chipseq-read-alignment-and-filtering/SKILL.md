---
name: chipseq-read-alignment-and-filtering
description: Use when starting from raw ChIP-seq FASTQ files (IP and matched control,
  e.g. KO transcription factor experiments from ENA accessions in Supplementary Table
  S1) that must be trimmed, aligned, and filtered before computing read coverage or
  fold enrichment over genomic intervals.
when_to_use_negative:
- Input is already a filtered BAM/aligned read set rather than raw FASTQ — skip directly
  to coverage computation with Bedtools
- The goal is peak calling or identifying new binding sites rather than assessing
  technical bias / false positive signal at predefined HOT regions in KO ChIP-seq
  data
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0102
tools:
- name: bbduk
  role: Trims adapters and low-quality bases from ChIP-seq reads before alignment
- name: Bowtie
  role: Aligns trimmed IP and control reads to the reference genome
- name: samtools
  role: Filters unmapped/multimapping reads and sorts and indexes the alignments
- name: Bedtools
  role: Computes read coverage over the HOT region intervals from the filtered alignments
- name: genomation
  role: R/Bioconductor package used to compute log2(IP/control) fold enrichment over
    HOT regions for each KO experiment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:27434c16721e2c718a5b7b994d79f49a97df01eb7dd9970419f1bcbca26b8cef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - Bowtie
  - bbduk
  - genomation
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chipseq-read-alignment-and-filtering

## Summary

A ChIP-seq preprocessing workflow (adapter trimming with bbduk, Bowtie alignment, samtools filtering/sorting/indexing) that turns raw IP and control FASTQ files into filtered BAM alignments ready for downstream quantification, e.g. computing IP/control enrichment over HOT regions in KO transcription factor experiments.

## When to use

Use when starting from raw ChIP-seq FASTQ files (IP and matched control, e.g. KO transcription factor experiments from ENA accessions in Supplementary Table S1) that must be trimmed, aligned, and filtered before computing read coverage or fold enrichment over genomic intervals.

## When NOT to use

- Input is already a filtered BAM/aligned read set rather than raw FASTQ — skip directly to coverage computation with Bedtools
- The goal is peak calling or identifying new binding sites rather than assessing technical bias / false positive signal at predefined HOT regions in KO ChIP-seq data

## Inputs

- Raw ChIP-seq FASTQ files (IP and matching control) downloaded from ENA per accessions in Supplementary Table S1
- Reference genome assembly for the studied species
- Genomic intervals of interest (HOT region BED/GFF intervals) for downstream coverage

## Outputs

- Trimmed FASTQ files (bbduk output)
- Filtered, sorted, and indexed BAM alignment files (samtools)
- Per-experiment read coverage over HOT region intervals (Bedtools)
- Per-experiment log2(IP/control) fold-enrichment values over HOT regions (genomation), written to ko_hot_enrichment.csv with enrichment labels

## How to apply

Download the KO transcription factor ChIP-seq IP and matching control fastq files from ENA using the accession numbers listed in Supplementary Table S1. Trim adapters and low-quality bases from the reads using bbduk, since residual adapter and low-quality bases degrade alignment. Align the trimmed reads to the reference genome using Bowtie, then use samtools to remove unmapped and multimapping reads, and sort and index the resulting alignments; this filtering step is essential for enrichment analysis because multimappers would inflate coverage at repetitive, GC-rich loci such as HOT regions. Pass the filtered BAMs to coverage quantification (Bedtools over the HOT region intervals) and fold-enrichment computation (genomation log2(IP/control) in R/Bioconductor) for each of the 25 KO experiments.

## Related tools

- **bbduk** (Trims adapters and low-quality bases from ChIP-seq reads before alignment)
- **Bowtie** (Aligns trimmed IP and control reads to the reference genome)
- **samtools** (Filters unmapped/multimapping reads and sorts and indexes the alignments)
- **Bedtools** (Computes read coverage over the HOT region intervals from the filtered alignments)
- **genomation** (R/Bioconductor package used to compute log2(IP/control) fold enrichment over HOT regions for each KO experiment)

## Examples

```
bbduk.sh in=IP.fastq out=trimmed.fastq; bowtie genome trimmed.fastq IP.sam; samtools view -F 4 IP.sam | samtools sort -o IP.bam; samtools index IP.bam
```

## Evaluation signals

- All 25 KO experiments produce a non-empty, indexed BAM for both IP and control after samtools filtering (unmapped and multimapping reads removed)
- ko_hot_enrichment.csv contains one log2(IP/control) row per KO experiment with a classification of clear enrichment vs. no enrichment
- More than half of the 25 KO ChIP-seq experiments show clear signal enrichment (IP/control) over HOT regions, consistent with the paper's reported false positives
- Experiments lacking enrichment should be checked for technical causes (low number of reads, low library complexity, unsuccessful IP) before labeling them true negatives

## Limitations

- A lack of signal in a KO ChIP-seq experiment might be caused by technical conditions such as low number of reads, low library complexity, or unsuccessful IP, so absence of enrichment is not definitive
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes
- The skill assesses predefined HOT regions from prior ChIP-seq compendia; it does not de novo identify binding sites

## Evidence

- [methods] workflow steps: [evidence span withheld — anchor status: not_found]
- [methods] filtering: [evidence span withheld — anchor status: not_found]
- [results] enrichment: [evidence span withheld — anchor status: not_found]
- [abstract] false positives: "Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions."
- [discussion] technical caveat: "a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP."
- [readme] repository: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
