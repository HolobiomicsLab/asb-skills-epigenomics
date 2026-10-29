---
name: chipseq-fold-enrichment-over-hot-regions
description: 'Use when assessing whether ChIP-seq signal at HOT regions is a technical
  artifact: you have KO transcription factor ChIP-seq IP/control FASTQ pairs and a
  HOT region interval set, and need per-experiment fold-enrichment over those regions.'
when_to_use_negative:
- When the goal is to identify genuine, functional transcription factor binding sites
  or canonical motif occupancy — HOT regions are defined by ChIP-seq and are shown
  to lack canonical motifs for the TFs thought to be bound there.
- When you need to decompose artifact signal into antibody-specific pull-down of non-target
  proteins versus direct binding to polynucleotide epitopes — the article states this
  proportion cannot be estimated with currently available data.
- When the ChIP-ed protein is present in the cell (non-KO experiments) — this skill's
  rationale depends on the target TF being absent so any signal is necessarily false
  positive.
edam_operation: http://edamontology.org/operation_0364
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: bbduk
  role: Trims adapters and low-quality bases from ChIP-seq reads before alignment
- name: Bowtie
  role: Aligns trimmed IP/control reads to the reference genome
- name: samtools
  role: Filters, sorts, and indexes alignments, removing unmapped and multi-mapping
    reads
- name: Bedtools
  role: Computes read coverage over the HOT region intervals
- name: genomation
  role: R/Bioconductor package used to compute log2(IP/control) fold enrichment over
    HOT regions per KO experiment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:6335dace4458901755cf010b5232a4d058730f4913bdf6ecc8734319ec8cfe61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - Bowtie
  - bbduk
  - genomation
  - samtools
  edam_operation: http://edamontology.org/operation_0364
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# chipseq-fold-enrichment-over-hot-regions

## Summary

Quantifies false-positive ChIP-seq signal on HOT (high-occupancy target) regions by computing log2(IP/control) fold enrichment for ChIP-seq experiments performed on knocked-out transcription factors, where the ChIP-ed protein is absent from the cell, following the 'HOT or not' analysis (NAR, 2019).

## When to use

Use when assessing whether ChIP-seq signal at HOT regions is a technical artifact: you have KO transcription factor ChIP-seq IP/control FASTQ pairs and a HOT region interval set, and need per-experiment fold-enrichment over those regions.

## When NOT to use

- When the goal is to identify genuine, functional transcription factor binding sites or canonical motif occupancy — HOT regions are defined by ChIP-seq and are shown to lack canonical motifs for the TFs thought to be bound there.
- When you need to decompose artifact signal into antibody-specific pull-down of non-target proteins versus direct binding to polynucleotide epitopes — the article states this proportion cannot be estimated with currently available data.
- When the ChIP-ed protein is present in the cell (non-KO experiments) — this skill's rationale depends on the target TF being absent so any signal is necessarily false positive.

## Inputs

- KO transcription factor ChIP-seq IP and matched control FASTQ files from ENA (accessions in Supplementary Table S1, 25 experiments)
- Reference genome FASTQ/index for Bowtie alignment
- HOT region interval file (BED-style genomic intervals)

## Outputs

- Filtered, sorted, indexed BAM alignment files for each IP and control
- Per-experiment log2(IP/control) fold enrichment values over HOT regions
- ko_hot_enrichment.csv with per-experiment enrichment values and clear-enrichment labels

## How to apply

Download the KO transcription factor ChIP-seq fastq files (IP and matching control) from ENA using accessions from Supplementary Table S1. Trim adapters and low-quality bases with bbduk, then align the trimmed reads to the reference genome with Bowtie. Filter, sort, and index alignments with samtools, removing unmapped and multi-mapping reads. Compute read coverage over the HOT region intervals using Bedtools, and use the genomation R/Bioconductor package to compute log2(IP/control) fold enrichment over HOT regions for each of the 25 KO experiments. Classify each experiment as showing clear enrichment on HOT regions and write the per-experiment enrichment values and labels to ko_hot_enrichment.csv; the article's claim is validated when more than half of the 25 KO ChIP-seq experiments show clear enrichment, demonstrating false positive signals at these loci.

## Related tools

- **bbduk** (Trims adapters and low-quality bases from ChIP-seq reads before alignment)
- **Bowtie** (Aligns trimmed IP/control reads to the reference genome)
- **samtools** (Filters, sorts, and indexes alignments, removing unmapped and multi-mapping reads)
- **Bedtools** (Computes read coverage over the HOT region intervals)
- **genomation** (R/Bioconductor package used to compute log2(IP/control) fold enrichment over HOT regions per KO experiment)

## Examples

```
bedtools coverage -a HOT_regions.bed -b IP.filtered.sorted.bam > IP_HOT_coverage.txt
```

## Evaluation signals

- More than half of the 25 KO ChIP-seq experiments are classified as showing clear signal enrichment (IP/control) over HOT regions
- ko_hot_enrichment.csv contains a log2(IP/control) enrichment value and an enrichment label for every one of the 25 KO experiments
- BAM files are filtered (unmapped and multi-mapping reads removed), sorted, and indexed before coverage computation
- Enrichment is computed as log2 of IP vs. matched control coverage over the same HOT region interval set, not raw IP signal alone
- Any KO experiment with no apparent enrichment should be checked against technical explanations (low read number, low library complexity, unsuccessful IP) before concluding absence of artifact

## Limitations

- A lack of signal in a KO ChIP-seq experiment might be caused by technical conditions such as a low number of reads, low library complexity, or unsuccessful IP, so non-enrichment does not by itself rule out artifact susceptibility.
- It is not possible with currently available data to estimate the proportion of antibody-specific error from pull-down of non-target proteins versus direct binding to polynucleotide epitopes.
- HOT regions are described as likely ChIP-seq artifacts similar to 'hyper-ChIPable' regions; the artificial enrichment is associated with features such as GC/CpG-rich k-mers, R-loops, and G-quadruplex DNA, so enrichment here indicates bias rather than true binding.

## Evidence

- [abstract] Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions.: "Using ChIP-seq data sets for knocked-out transcription factors, we demonstrate presence of false positive signals on HOT regions."
- [abstract] HOT regions are likely ChIP-seq artifacts: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [discussion] lack of signal caveat: "a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP."
- [readme] repository scripts note: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [abstract] motif absence context: "Despite these features, HOT regions are solely defined using ChIP-seq experiments and shown to lack canonical motifs for transcription factors that are thought to be bound there."
