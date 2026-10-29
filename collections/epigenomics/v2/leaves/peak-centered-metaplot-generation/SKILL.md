---
name: peak-centered-metaplot-generation
description: Use when you need a standardized, reproducible averaged profile of sequencing
  signal (BAM/SAM ChIP-seq or CUT&RUN reads) aggregated around a set of peak centers
  to compare conditions, such as WT vs KO ChIP levels.
when_to_use_negative:
- 'The goal is genome-browser track files: generation of genome-wide profiles in bigwig
  format is not supported and deferred to future versions of SeqCode'
- The aggregation unit should be genes rather than peaks (use produceTSSplots/produceGENEplots
  instead of peak-centered plots)
- The web tool is intended as a teaching resource with no limits locally; very large
  input sets should be processed with the standalone command line, not the SeqCode
  website
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: SeqCode
  role: Runs recoverChIPlevels for depth-normalized peak counts and producePEAKplots
    to build the peak-centered meta-plots, rendering graphics through internal R scripts
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Maps raw ChIP-seq reads to the reference genome (mm9/mm10) prior to peak-centered
    plotting
- name: MACS
  role: Performs peak calling on ChIP-seq experiments to supply the reference peak
    list centered in the meta-plot
- name: SAMtools
  role: Source code integrated into SeqCode (with BCFtools and HTSlib) to read the
    input BAM/SAM files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:39e277ee78e7d1466569d37c63cf934bd85e08b9de9a244a7a3a98e0b3fb2ba2
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-centered-metaplot-generation

## Summary

Generate high-quality averaged meta-plots centered on a reference peak list (e.g. MACS-called ChIP-seq peaks) with SeqCode's producePEAKplots/producePEAKmaps, quantifying ChIP signal (e.g. MLL2, H3K4me3) in WT vs perturbation conditions and rendering the plots via internal R scripts.

## When to use

Use when you need a standardized, reproducible averaged profile of sequencing signal (BAM/SAM ChIP-seq or CUT&RUN reads) aggregated around a set of peak centers to compare conditions, such as WT vs KO ChIP levels.

## When NOT to use

- The goal is genome-browser track files: generation of genome-wide profiles in bigwig format is not supported and deferred to future versions of SeqCode
- The aggregation unit should be genes rather than peaks (use produceTSSplots/produceGENEplots instead of peak-centered plots)
- The web tool is intended as a teaching resource with no limits locally; very large input sets should be processed with the standalone command line, not the SeqCode website

## Inputs

- Chromosome size list and RefSeq gene/transcript annotation files for the genome assembly (e.g. from UCSC, ChromInfo.txt and refGene.txt)
- Mapped reads in BAM or SAM format (unaligned reads excluded)
- Peak/region coordinate set from MACS peak calling of the ChIP-seq experiment
- Raw ChIP-seq FASTQ files (e.g. GSM2645501 MLL2 WT, GSM2645502 MLL2 KO, GSM2645495/GSM2645496 H3K4me3 WT/KO from NCBI GEO)

## Outputs

- Averaged peak-centered meta-plot (PDF) per sample, produced by internal R scripts
- Output folder containing the plot PDF, the R script that generated it, and the underlying data files
- Average, maximum and total read counts over peaks from recoverChIPlevels, normalized by sequencing depth

## How to apply

Map raw FASTQ reads to the reference genome (e.g. mm9/mm10 with Bowtie), exclude unaligned reads from the resulting BAM files since SeqCode internally reads BAM/SAM via integrated SAMtools/HTSlib, and call peaks with MACS to define the reference region set. Run SeqCode recoverChIPlevels to compute average, maximum and total read counts over the peaks, normalized by sequencing depth (or by the spike-in genome read count when spike-in material such as MTF2 samples is included, with the factor introduced manually). Then run producePEAKplots to generate the averaged meta-plot centered on the peak list; the command writes an output folder containing the resulting PDF plot, the R script used to render it, and the data files (customizable with -x prefix, verbose mode with -v). Judge correctness by checking the output PDF exists and is a valid plot, and that relative signal levels across conditions (e.g. MLL2 and H3K4me3 lower in KO than WT) qualitatively match expectations.

## Related tools

- **SeqCode** (Runs recoverChIPlevels for depth-normalized peak counts and producePEAKplots to build the peak-centered meta-plots, rendering graphics through internal R scripts) — https://github.com/eblancoga/seqcode
- **Bowtie** (Maps raw ChIP-seq reads to the reference genome (mm9/mm10) prior to peak-centered plotting)
- **MACS** (Performs peak calling on ChIP-seq experiments to supply the reference peak list centered in the meta-plot)
- **SAMtools** (Source code integrated into SeqCode (with BCFtools and HTSlib) to read the input BAM/SAM files)

## Examples

```
./tests/test_producePEAKplots.pl -v
```

## Evaluation signals

- Output PDF plot file exists and passes a file_format check in the producePEAKplots output folder
- Meta-plot is centered on the MACS peak list and shows the expected averaged profile shape around peak centers
- recoverChIPlevels values (average, maximum, total reads) decrease consistently in the perturbation condition (e.g. MLL2 and H3K4me3 lower in MLL2-KO mESCs than WT), qualitatively matching the reported pattern
- BAM inputs contain no unaligned reads, since these must be excluded before SeqCode processing
- Normalization is by sequencing depth (total reads in the BAM), or by the manually supplied spike-in factor when spike-in material was included

## Limitations

- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Spike-in normalization factor must be manually introduced by users
- The SeqCode web tools are intended as a teaching resource; the command line has no input size limits, but the website does

## Evidence

- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10): "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] MACS was used to perform the peak calling: "MACS12 was used to perform the peak calling of the ChIP-seq experiments."
- [methods] SeqCode normalization by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Unaligned reads must be excluded from the resulting BAM files: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] Output folder contains plots, R scripts and data files: "The output of the program is usually stored into an output folder in which the user will find the resulting plots, the scripts to generate the plots in R (if any), and the data files."
