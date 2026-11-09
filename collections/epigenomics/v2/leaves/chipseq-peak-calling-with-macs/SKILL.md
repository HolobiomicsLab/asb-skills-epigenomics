---
name: chipseq-peak-calling-with-macs
description: Use when mapped ChIP-seq reads (BAM/SAM, e.g. Bowtie-aligned to mm9/mm10)
  must be reduced to a peak set that serves as the region reference for meta-plots,
  heatmaps, or ChIP-level quantification in SeqCode-based workflows.
when_to_use_negative:
- The analysis goal is genome-browser track generation in bigWig format — the article
  states this is not supported and deferred to future SeqCode versions
- The input is raw FASTQ that has not yet been mapped with Bowtie and cleaned of unaligned
  reads; peak calling requires aligned BAM/SAM input
- Quantification is needed over arbitrary genes/TSS windows rather than MACS-defined
  peak regions (SeqCode TSS/GENE commands fit better)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: MACS
  role: Peak caller that produces the reference peak/region set used by SeqCode producePEAKplots
    and recoverChIPlevels
- name: Bowtie
  role: Aligns raw ChIP-seq FASTQ reads to the mouse genome (mm9/mm10) upstream of
    peak calling
- name: SeqCode
  role: Consumes MACS peak lists and cleaned BAM files to produce peak-centered meta-plots
    and depth-normalized ChIP-level counts
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Integrated inside SeqCode to read BAM/SAM files; unaligned reads must be excluded
    from BAMs before SeqCode processing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:819db7ffbfd262d63144ac1e13acc1aa84251d15e8a7fae24c62515a0d007b8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - MACS
  - MACS2
  - R
  - SAMtools
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- chipseq-peak-calling
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chipseq-peak-calling-with-macs

## Summary

Perform peak calling on ChIP-seq experiments with MACS to obtain the reference set of enriched regions that downstream SeqCode visualization and quantification commands (e.g. producePEAKplots, recoverChIPlevels) operate on, as demonstrated in the WT vs MLL2-KO mESC comparison of MLL2 and H3K4me3.

## When to use

Use when mapped ChIP-seq reads (BAM/SAM, e.g. Bowtie-aligned to mm9/mm10) must be reduced to a peak set that serves as the region reference for meta-plots, heatmaps, or ChIP-level quantification in SeqCode-based workflows.

## When NOT to use

- The analysis goal is genome-browser track generation in bigWig format — the article states this is not supported and deferred to future SeqCode versions
- The input is raw FASTQ that has not yet been mapped with Bowtie and cleaned of unaligned reads; peak calling requires aligned BAM/SAM input
- Quantification is needed over arbitrary genes/TSS windows rather than MACS-defined peak regions (SeqCode TSS/GENE commands fit better)

## Inputs

- BAM files of Bowtie-mapped ChIP-seq reads with unaligned reads excluded (e.g. GSM2645501 MLL2 WT, GSM2645502 MLL2 KO, GSM2645495 H3K4me3 WT, GSM2645496 H3K4me3 KO)
- Chromosome size list and RefSeq gene transcript annotation files for the genome assembly (e.g. mm9/mm10) retrieved from the UCSC genome browser
- Optional spike-in genome read counts for manual spike-in normalization

## Outputs

- MACS peak list (reference regions/regions file) for the ChIP-seq experiment
- Peak-centered meta-plots from SeqCode producePEAKplots (PDF, R-generated) showing averaged ChIP signal per sample
- ChIP-level statistics from recoverChIPlevels (average, maximum, total read counts per peak, normalized by sequencing depth)
- Composite figures such as fig7a_mll2_h3k4me3_wt_ko.pdf reproducing article layouts

## How to apply

After mapping raw ChIP-seq FASTQ reads to the target genome (mouse mm9/mm10 in the article's examples) with Bowtie, exclude unaligned reads from the resulting BAM files since SeqCode commands require cleaned BAMs (SAMtools/HTSlib are integrated internally to read them). Run MACS on the ChIP-seq samples to perform peak calling and obtain the reference peak/region set; this peak list becomes the coordinate input for SeqCode producePEAKplots (averaged meta-plots centered on peak centers) and recoverChIPlevels (average, maximum, and total read counts normalized by sequencing depth). For experiments with spike-in material (e.g. the MTF2 samples in Fig. 7), replace the sequencing-depth normalization factor with the number of reads mapped over the spike-in genome, which must be manually introduced by the user. In the MLL2 example, peaks called on GSM2645501/2645502 (MLL2 WT/KO) and GSM2645495/2645496 (H3K4me3 WT/KO) support the composite figure showing signal loss upon KO.

## Related tools

- **MACS** (Peak caller that produces the reference peak/region set used by SeqCode producePEAKplots and recoverChIPlevels)
- **Bowtie** (Aligns raw ChIP-seq FASTQ reads to the mouse genome (mm9/mm10) upstream of peak calling)
- **SeqCode** (Consumes MACS peak lists and cleaned BAM files to produce peak-centered meta-plots and depth-normalized ChIP-level counts) — https://github.com/eblancoga/seqcode
- **SAMtools** (Integrated inside SeqCode to read BAM/SAM files; unaligned reads must be excluded from BAMs before SeqCode processing)

## Examples

```
bin/producePEAKplots ChromInfo.txt refGene.txt MLL2_WT.bam MLL2_KO.peaks mll2_wt_ko 5000
```

## Evaluation signals

- MACS completes on the cleaned BAM and emits a non-empty peak list for each ChIP-seq sample (MLL2 WT/KO, H3K4me3 WT/KO)
- SeqCode producePEAKplots output folder contains the expected peak-centered meta-plot PDFs plus the R scripts used to generate them
- recoverChIPlevels returns average, maximum, and total read counts per peak, and values are normalized by sequencing depth (total reads in the BAM)
- Plotted MLL2 and H3K4me3 signals are visibly lower in KO than WT, qualitatively matching Fig. 7a of the article
- The composite figure (e.g. fig7a_mll2_h3k4me3_wt_ko.pdf) passes file_exists and file_format checks on the PDF

## Limitations

- Spike-in normalization factors must be manually introduced by the user into SeqCode applications; they are not computed automatically
- Unaligned reads must be excluded from BAM files before SeqCode commands will process them
- Generation of genome-wide profiles in bigWig format is not supported in the current SeqCode version
- Peak-calling quality depends on upstream Bowtie mapping choices (genome assembly mm9 vs mm10)

## Evidence

- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] MACS12 was used to perform the peak calling of the ChIP-seq experiments.: "MACS12 was used to perform the peak calling of the ChIP-seq experiments."
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Raw data of the examples introduced throughout the article (Table 4) were retrieved from the NCBI GEO repository74.: "Raw data of the examples introduced throughout the article (Table 4) were retrieved from the NCBI GEO repository74."
