---
name: boxplot-visualization-of-genomic-signal
description: Use when you have sequencing-depth-normalized ChIP-seq (or similar) read-count
  levels computed over gene bodies and need to compare signal distributions across
  categorical gene sets (e.g. expression classes) with publication-quality boxplots.
when_to_use_negative:
- Input is already a processed feature-count or expression table with no underlying
  BAM/SAM mapping — SeqCode recoverChIPlevels requires mapped reads plus genome/annotation
  files
- You need genome-wide signal profiles in bigWig format for genome browser tracks
  — the paper states bigwig generation is not supported and deferred to future versions
- The web tools at ldicrocelab.crg.es are intended as a teaching resource on a selection
  of published ChIP-seq samples; large local data sets must go through the command-line
  release
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: 'Core toolkit: recoverChIPlevels computes sequencing-depth-normalized avg/max/total
    read counts over gene bodies per group; R-backed plotting produces the boxplots'
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Maps ChIP-seq reads to the reference genome (mm9) before count recovery
- name: MACS
  role: Peak caller applied to ChIP-seq experiments (MACS12); outputs can be cleaned
    via SeqCode's processmacs step
- name: SAMtools
  role: Internally integrated by SeqCode to read BAM and SAM mapped-read inputs
- name: R
  role: Generates the final boxplot images internally within SeqCode commands
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:ae5e82713df8bd1506ce50d4fd43ec2a8035f46f6a9f18de017cb3f9f86fcbb0
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# boxplot-visualization-of-genomic-signal

## Summary

Use SeqCode's recoverChIPlevels plus its R-based plotting to produce boxplots of normalized ChIP-seq signal intensities (avg/max/total read counts) across predefined gene groups, e.g. highly, moderately, and silently expressed RefSeq genes, as in Fig. 6 of the SeqCode paper correlating active/repressive chromatin marks with expression classes.

## When to use

Use when you have sequencing-depth-normalized ChIP-seq (or similar) read-count levels computed over gene bodies and need to compare signal distributions across categorical gene sets (e.g. expression classes) with publication-quality boxplots.

## When NOT to use

- Input is already a processed feature-count or expression table with no underlying BAM/SAM mapping — SeqCode recoverChIPlevels requires mapped reads plus genome/annotation files
- You need genome-wide signal profiles in bigWig format for genome browser tracks — the paper states bigwig generation is not supported and deferred to future versions
- The web tools at ldicrocelab.crg.es are intended as a teaching resource on a selection of published ChIP-seq samples; large local data sets must go through the command-line release

## Inputs

- Mapped sequencing reads in BAM/SAM format (unaligned reads excluded)
- ChromInfo.txt: list of chromosome names and sizes for the genome assembly (e.g. mm9), from UCSC
- refGene.txt: RefSeq transcript annotation file for the assembly
- Gene-group definition file (e.g. 100 highly, 100 moderately, 100 silently expressed RefSeq genes derived from RNA-seq GSM1562339)
- Optionally MACS12 peak-calling outputs (BedGraph profiles) cleaned via processmacs

## Outputs

- Per-group normalized ChIP levels table (avg/max/total read counts over gene bodies, sequencing-depth normalized)
- Boxplot PDF figure comparing signal intensity distributions across gene groups (Fig. 6-style)
- R scripts and data files stored in the result output folder for reproducibility

## How to apply

Retrieve raw ChIP-seq FASTQ files from NCBI GEO (e.g. H3K4me3 SRX367147, H3K27me3 GSM1019772), map reads to mm9 with Bowtie, exclude unaligned reads from the BAM files, and call peaks with MACS12. Preprocess MACS outputs with the SeqCode processmacs filter step if using BedGraph profiles. Run SeqCode recoverChIPlevels with the UCSC-derived mm9 chromInfo and refGene annotation files to compute avg/max/total normalized read counts (normalized by sequencing depth, i.e. total reads in the BAM) over gene bodies of each expression group (e.g. three groups of 100 highly, moderately, and silently expressed genes selected from RNA-seq data GSM1562339). Generate the per-group boxplots with SeqCode's R-based plotting engine, which stores the PDF plots plus the underlying R scripts and data files in an output folder. Judge success by qualitative concordance with the expected pattern (active marks like H3K4me3, H3K36me3, Ser5P, Ser2P high on highly expressed genes; Polycomb marks like H3K27me3, Suz12, Jarid2, Ring1b, H2Aub high on silent genes), as no numeric threshold is specified.

## Related tools

- **SeqCode** (Core toolkit: recoverChIPlevels computes sequencing-depth-normalized avg/max/total read counts over gene bodies per group; R-backed plotting produces the boxplots) — https://github.com/eblancoga/seqcode
- **Bowtie** (Maps ChIP-seq reads to the reference genome (mm9) before count recovery)
- **MACS** (Peak caller applied to ChIP-seq experiments (MACS12); outputs can be cleaned via SeqCode's processmacs step)
- **SAMtools** (Internally integrated by SeqCode to read BAM and SAM mapped-read inputs)
- **R** (Generates the final boxplot images internally within SeqCode commands)

## Examples

```
./tests/test_recoverChIPlevels.pl -v
```

## Evaluation signals

- Boxplot figure reproduces the Fig. 6 qualitative pattern: active marks (H3K4me3, H3K36me3, Ser5P, Ser2P) signal positively correlates with expression; Polycomb/repressive marks (H3K27me3, Suz12, Jarid2, Ring1b, H2Aub) high on silent genes
- ChIP levels are normalized by sequencing depth (total reads in the BAM file); spike-in-normalized samples use mapped spike-in-genome read counts instead
- Each recoverChIPlevels run creates an output folder containing the data files plus the R scripts that generated the plots
- SeqCode tests/ folder Perl scripts (e.g. test_recoverChIPlevels.pl) verify correct binary generation and successful termination with [OK]/[DONE] markers on output PDFs

## Limitations

- BigWig genome-wide profile generation is unsupported and deferred to future versions
- Spike-in normalization factors must be manually introduced by the user
- The SeqCode web portal is a teaching resource limited to a selection of published ChIP-seq samples; the command-line release has no size limits but requires local installation via 'make all' on UNIX platforms (Linux/macOS)

## Evidence

- [methods] recoverChIPlevels computes normalized counts: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] MACS peak calling: "MACS12 was used to perform the peak calling of the ChIP-seq experiments."
- [methods] unaligned reads excluded: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] R-based graphics inside SeqCode: "Meta-plots, heatmaps, boxplots and the rest of images produced by SeqCode are internally generated using R."
- [readme] boxplot application list: "Generate boxplots with multiple graphical options to customize the final result"
