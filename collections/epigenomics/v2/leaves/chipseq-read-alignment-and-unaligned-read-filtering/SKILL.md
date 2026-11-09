---
name: chipseq-read-alignment-and-unaligned-read-filtering
description: Use when starting from raw ChIP-seq FASTQ files (e.g. GEO accessions
  such as GSM2645501/GSM2645502) that must be mapped to a genome (mm9, mm10, hg38,
  dm3) and converted into clean BAM files for peak calling or SeqCode read-count/visualization
  commands.
when_to_use_negative:
- Input is already a mapped BAM/SAM file — only the unaligned-read exclusion applies,
  no Bowtie mapping is needed
- Data are RNA-seq or ATAC-seq expression/count matrices where sequencing data are
  already processed past the mapping stage and only visualization (SeqCode meta-plots,
  heatmaps, boxplots) is required
- A genome assembly not supported by Bowtie/RefSeq annotations is required, since
  SeqCode relies on RefSeq reference annotations for every organism
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Bowtie
  role: Aligns ChIP-seq reads to the reference genome (mm9, mm10, hg38, dm3) to produce
    the initial alignment from which BAM files are derived
- name: SeqCode
  role: Consumes the filtered BAM/SAM files downstream; internally integrates SamTools
    and HTS C libraries to read BAM files and computes ChIP levels and plots
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Library integrated inside SeqCode (with BCFtools and HTSlib) to efficiently
    read the mapped BAM/SAM files; unaligned reads must be excluded beforehand
- name: MACS
  role: Performs peak calling on the aligned, filtered ChIP-seq BAM files to obtain
    the reference peak/region set for visualization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:37d34742e8d5196adf654b9c61a9865019ea1c342f9e57c1ab21630068c58608
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - MACS
  - R
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chipseq-read-alignment-and-unaligned-read-filtering

## Summary

Maps ChIP-seq FASTQ reads to a reference genome with Bowtie (e.g. mouse mm9/mm10) and removes unaligned reads from the resulting BAM files so they are valid inputs for SeqCode commands and downstream peak calling. This preprocessing step underpins reproducible ChIP-level quantification and visualization in SeqCode workflows.

## When to use

Use when starting from raw ChIP-seq FASTQ files (e.g. GEO accessions such as GSM2645501/GSM2645502) that must be mapped to a genome (mm9, mm10, hg38, dm3) and converted into clean BAM files for peak calling or SeqCode read-count/visualization commands.

## When NOT to use

- Input is already a mapped BAM/SAM file — only the unaligned-read exclusion applies, no Bowtie mapping is needed
- Data are RNA-seq or ATAC-seq expression/count matrices where sequencing data are already processed past the mapping stage and only visualization (SeqCode meta-plots, heatmaps, boxplots) is required
- A genome assembly not supported by Bowtie/RefSeq annotations is required, since SeqCode relies on RefSeq reference annotations for every organism

## Inputs

- Raw ChIP-seq FASTQ files (e.g. GSM2645501 MLL2 WT, GSM2645502 MLL2 KO, GSM2645495 H3K4me3 WT, GSM2645496 H3K4me3 KO from NCBI GEO)
- Reference genome assembly index (e.g. mm9 or mm10) for Bowtie
- List of chromosomes and their sizes plus RefSeq gene transcript annotations (retrievable from the UCSC genome browser) for subsequent SeqCode commands

## Outputs

- Per-sample BAM files with unaligned reads removed
- Read counts per BAM file used for sequencing-depth normalization
- Mapped reads suitable for MACS peak calling and SeqCode recoverChIPlevels/producePEAKplots analysis

## How to apply

Retrieve raw ChIP-seq FASTQ files from NCBI GEO, then map reads with Bowtie over the chosen genome assembly (the article used mouse mm9/mm10, human hg38, and fruit fly dm3). Unaligned reads must be excluded from the resulting BAM files, because SeqCode commands require BAM/SAM input and internally integrate the SamTools/HTS C libraries to read them efficiently. Use the filtered, per-sample BAM files as the reference for MACS peak calling and for downstream SeqCode steps (recoverChIPlevels, producePEAKplots), where values are normalized by sequencing depth (total number of reads in the BAM file). Validate the step by checking that each sample yields a valid BAM containing only aligned reads and that read counts used for depth normalization are consistent with the mapped reads.

## Related tools

- **Bowtie** (Aligns ChIP-seq reads to the reference genome (mm9, mm10, hg38, dm3) to produce the initial alignment from which BAM files are derived)
- **SeqCode** (Consumes the filtered BAM/SAM files downstream; internally integrates SamTools and HTS C libraries to read BAM files and computes ChIP levels and plots) — https://github.com/eblancoga/seqcode
- **SAMtools** (Library integrated inside SeqCode (with BCFtools and HTSlib) to efficiently read the mapped BAM/SAM files; unaligned reads must be excluded beforehand)
- **MACS** (Performs peak calling on the aligned, filtered ChIP-seq BAM files to obtain the reference peak/region set for visualization)

## Examples

```
bowtie -S mm10 reads.fastq | samtools view -b -F 4 > aligned_reads.bam
```

## Evaluation signals

- BAM files exist per sample (WT and KO for each ChIP target) and contain no unaligned reads, as required by SeqCode commands
- Each BAM is readable by the SeqCode/SamTools HTSlib integration without errors (e.g. recoverChIPlevels runs and reports read counts)
- Bowtie mapping was performed against the correct genome assembly matching the downstream RefSeq annotation (mm9/mm10 consistency)
- Total read counts per BAM used for sequencing-depth normalization are consistent with the number of mapped (aligned) reads
- Downstream MACS peak calling succeeds on the filtered BAMs, producing a usable peak/region set for producePEAKplots meta-plots

## Limitations

- Unaligned reads are not dropped automatically by SeqCode; the article states they 'must be excluded' from BAM files before processing, so forgetting this step breaks downstream commands
- Spike-in normalization requires the spike-in factor to be manually introduced by the user rather than computed automatically
- Generation of genome-wide profiles in bigwig format is not supported and is deferred to future versions of SeqCode

## Evidence

- [methods] Bowtie mapping: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] Unaligned read exclusion: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] SamTools/HTSlib integration: "Our programs internally integrate the SamTools and HTS C libraries18 in order to efficiently read BAM and SAM files."
- [methods] Depth normalization: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [readme] BAM/SAM inputs in README: "Commands working with sequencing data require the user to provide the mapped reads of the experiment (BAM or SAM format)."
