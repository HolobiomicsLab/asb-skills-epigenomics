---
name: chromatin-factor-wt-vs-knockout-signal-comparison
description: Use when comparing ChIP-seq signal of a chromatin regulator (e.g. MLL2)
  or its histone mark (e.g. H3K4me3) between WT and genetic-knockout samples, and
  you need depth-normalized ChIP levels plus averaged peak-centered meta-plots and
  boxplots.
when_to_use_negative:
- You need genome-wide continuous bigWig signal tracks — the paper explicitly defers
  bigwig generation to future SeqCode versions
- Differential binding analysis requiring statistical significance calls (e.g. peak-level
  p-values between conditions) — this skill produces normalized levels and qualitative
  visual comparison, not statistical tests
- Input is RNA-seq expression data rather than ChIP-seq signal over peaks — use gene-based
  SeqCode plots (produceTSSplots/produceGENEplots) instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: recoverChIPlevels computes depth-normalized read counts; producePEAKplots
    generates peak-centered meta-plots; R scripts produce boxplots
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: maps ChIP-seq reads to the mouse genome (mm9/mm10) prior to SeqCode processing
- name: MACS
  role: peak calling of the ChIP-seq experiments to define the reference peak/region
    set
- name: SAMtools
  role: integrated internally by SeqCode (with HTSlib) to read BAM files; used to
    remove unaligned reads
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:f1ed34c1ddb89ff144222e4bf72e1b9a204b3b2d905fc75877e04d88cce17dd7
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromatin-factor-wt-vs-knockout-signal-comparison

## Summary

Reproduce the WT vs MLL2-KO mESC ChIP-seq comparison (Mas et al., Fig. 7a) showing that genetic deletion of MLL2 erases MLL2 ChIP-seq occupancy and correspondingly reduces its H3K4me3 product, using SeqCode for read counting, normalization and meta-plot/boxplot visualization.

## When to use

Use when comparing ChIP-seq signal of a chromatin regulator (e.g. MLL2) or its histone mark (e.g. H3K4me3) between WT and genetic-knockout samples, and you need depth-normalized ChIP levels plus averaged peak-centered meta-plots and boxplots.

## When NOT to use

- You need genome-wide continuous bigWig signal tracks — the paper explicitly defers bigwig generation to future SeqCode versions
- Differential binding analysis requiring statistical significance calls (e.g. peak-level p-values between conditions) — this skill produces normalized levels and qualitative visual comparison, not statistical tests
- Input is RNA-seq expression data rather than ChIP-seq signal over peaks — use gene-based SeqCode plots (produceTSSplots/produceGENEplots) instead

## Inputs

- Raw ChIP-seq FASTQ files for GSM2645501 (MLL2 WT), GSM2645502 (MLL2 KO), GSM2645495 (H3K4me3 WT), GSM2645496 (H3K4me3 KO) from NCBI GEO (Mas et al.)
- BAM files of mapped reads per sample (unaligned reads removed), mouse genome assembly mm9 or mm10
- MACS peak list (reference peak/region set) for producePEAKplots
- Chromosome-size (ChromInfo.txt) and RefSeq gene annotation (refGene.txt) files for the analyzed assembly

## Outputs

- Per-sample MLL2 and H3K4me3 ChIP-level values (average, maximum, total counts) normalized by sequencing depth
- Averaged meta-plots (PDF) centered on the MACS peak list for each of the four samples (producePEAKplots)
- Boxplots of ChIP signal intensity (PDF) generated with the SeqCode R-based graphics
- Composite figure fig7a_mll2_h3k4me3_wt_ko.pdf reproducing the Fig. 7a layout

## How to apply

Retrieve raw ChIP-seq FASTQ files for GSM2645501 (MLL2 WT), GSM2645502 (MLL2 KO), GSM2645495 (H3K4me3 WT) and GSM2645496 (H3K4me3 KO) from NCBI GEO, then map reads to mouse mm9/mm10 with Bowtie, excluding unaligned reads from the BAM files since SeqCode cannot process them. Call peaks with MACS to define the reference peak/region set. Run SeqCode recoverChIPlevels to compute average, maximum and total read counts of MLL2 and H3K4me3 in WT and KO, normalizing each value by sequencing depth (total reads in the BAM); if spike-in material is present, supply the spike-in factor manually. Run SeqCode producePEAKplots to generate averaged meta-plots centered on the MACS peak list for each of the four samples, and generate boxplots of ChIP levels via the SeqCode R-based graphics. Assemble the plots into a composite figure (fig7a_mll2_h3k4me3_wt_ko.pdf) mirroring the layout of Fig. 7a.

## Related tools

- **SeqCode** (recoverChIPlevels computes depth-normalized read counts; producePEAKplots generates peak-centered meta-plots; R scripts produce boxplots) — https://github.com/eblancoga/seqcode
- **Bowtie** (maps ChIP-seq reads to the mouse genome (mm9/mm10) prior to SeqCode processing)
- **MACS** (peak calling of the ChIP-seq experiments to define the reference peak/region set)
- **SAMtools** (integrated internally by SeqCode (with HTSlib) to read BAM files; used to remove unaligned reads)

## Examples

```
bin/produceTSSmaps tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_1 5000
```

## Evaluation signals

- file_exists and file_format checks pass on fig7a_mll2_h3k4me3_wt_ko.pdf
- Plotted MLL2 ChIP-seq signal is visibly lower in KO than in WT in both meta-plots and boxplots
- H3K4me3 signal shows a corresponding decrease in KO relative to WT, qualitatively matching Fig. 7a
- recoverChIPlevels output tables report values normalized by sequencing depth (total reads in each BAM file)
- Input BAM files contain no unaligned reads before SeqCode processing

## Limitations

- Spike-in normalization factor must be manually introduced by users (relevant if spike-in material is used instead of depth normalization)
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Unaligned reads must be excluded from BAM files before SeqCode processing or commands will fail
- Comparison is qualitative; no statistical significance testing of WT vs KO differences is part of the workflow

## Evidence

- [methods] Bowtie was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] Normalization of values by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Exclusion of unaligned reads from BAM files: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] Meta-plots, heatmaps and boxplots generated with R: "Meta-plots, heatmaps, boxplots and the rest of images produced by SeqCode are internally generated using R."
