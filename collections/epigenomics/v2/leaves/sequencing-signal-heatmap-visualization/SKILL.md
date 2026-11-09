---
name: sequencing-signal-heatmap-visualization
description: Use when you have BAM/SAM reads mapped to a genome with a RefSeq refGene
  annotation and want standardized TSS-centered heatmaps or aggregated meta-plots
  of ChIP-seq or RNA-seq signal.
when_to_use_negative:
- You need genome-wide continuous profiles in bigWig format — the article states this
  is not yet supported and deferred to future versions
- Your reads are not yet mapped — quality control, read mapping (e.g. Bowtie) and
  exclusion of unaligned reads must be completed first
- You rely on annotations outside the RefSeq catalog — SeqCode relies on RefSeq reference
  annotations
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: SeqCode
  role: Runs produceTSSmaps/produceTSSplots to compute TSS-centered heatmaps and meta-plots
    from BAM reads and RefSeq annotations, and emits the R scripts that render the
    graphics
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Internally integrated into SeqCode (BCFtools/SAMtools/HTSlib) to read BAM/SAM
    inputs; unaligned reads must be excluded from BAMs before SeqCode commands run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:f5a1a88d41e0f7c0b6e30f5244950b6ad60eb04da937ece7428cf38ec434f7e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# sequencing-signal-heatmap-visualization

## Summary

Generate TSS-centered sequencing signal heatmaps and meta-plots (e.g. H3K4me3 ChIP-seq in mESC) with SeqCode's produceTSSmaps and produceTSSplots, rendering the final graphics via the R scripts emitted into the output folder.

## When to use

Use when you have BAM/SAM reads mapped to a genome with a RefSeq refGene annotation and want standardized TSS-centered heatmaps or aggregated meta-plots of ChIP-seq or RNA-seq signal.

## When NOT to use

- You need genome-wide continuous profiles in bigWig format — the article states this is not yet supported and deferred to future versions
- Your reads are not yet mapped — quality control, read mapping (e.g. Bowtie) and exclusion of unaligned reads must be completed first
- You rely on annotations outside the RefSeq catalog — SeqCode relies on RefSeq reference annotations

## Inputs

- Chromosome-size file (e.g. UCSC chromInfo.txt) for the genome assembly
- RefSeq refGene.txt transcript annotation file
- Mapped reads in BAM or SAM format with unaligned reads removed (e.g. H3K4me3_chr10.bam)
- Target gene list (e.g. genes_chr10.txt)
- Window size parameter (default 5000 bp around the TSS; -w 1000 for lower resolution)

## Outputs

- Output folder with TSS heatmap data and PlotHEATmap PDF (e.g. PlotHEATmap_test_1_5000.pdf)
- R script (Rscript_produceTSSmaps.txt) for regenerating the graphics
- Averaged TSS meta-plot PDF when using produceTSSplots

## How to apply

Install SeqCode from the GitHub distribution with 'make all', which builds the SAMtools/HTSlib libraries and the bin/ applications. Provide the chromosome-size file (ChromInfo.txt from UCSC), the RefSeq refGene.txt annotations, the aligned reads as BAM/SAM (unaligned reads excluded with SAMtools beforehand), and a target gene list; run produceTSSmaps over a 5000 bp window around the TSS to obtain the heatmap, or produceTSSplots for the averaged meta-plot. If memory or graphics quality must be tuned, adjust the binning window with the -w option (e.g. -w 1000 for lower resolution); the article shows results remain biologically consistent across window resolutions, with only graphical quality and memory affected. Each command creates an output folder containing the data files plus an R script (Rscript_produceTSSmaps.txt) that must be executed (e.g. R CMD BATCH) to render the final PlotHEATmap PDF.

## Related tools

- **SeqCode** (Runs produceTSSmaps/produceTSSplots to compute TSS-centered heatmaps and meta-plots from BAM reads and RefSeq annotations, and emits the R scripts that render the graphics) — https://github.com/eblancoga/seqcode
- **SAMtools** (Internally integrated into SeqCode (BCFtools/SAMtools/HTSlib) to read BAM/SAM inputs; unaligned reads must be excluded from BAMs before SeqCode commands run)

## Examples

```
bin/produceTSSmaps tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_1 5000
```

## Evaluation signals

- A PlotHEATmap PDF is produced for every window setting inside the output folder (e.g. PlotHEATmap_test_1_5000.pdf, PlotHEATmap_test_2_5000.pdf)
- Heatmap signal patterns around the TSS remain consistent when re-run at a lower resolution (-w 1000) versus the default 5000 bp window, matching Supplementary Figure 6
- The output folder contains the R script and data files alongside the final PDF, confirming the two-stage rendering completed
- The test suite terminates successfully (tests/test_produceTSSmaps.pl checks all seven output PDFs and reports 'Successful termination: [DONE]')

## Limitations

- Only lower-resolution settings (-w) reduce graphical quality; coarser windows are not recommended for publication-grade figures
- Generation of genome-wide profiles in bigWig format is not supported and deferred to future versions
- The SeqCode web version is intended as a teaching resource; the command line release has no limits on input set or size
- No formal changelog is provided for the software, so version tracking of graphical behavior is manual

## Evidence

- [methods] Our programs internally integrate the SamTools and HTS C libraries18 in order to efficiently read BAM and SAM files.: "Our programs internally integrate the SamTools and HTS C libraries18 in order to efficiently read BAM and SAM files."
- [methods] Chromosomes are divided into series of consecutive fragments of the same size, which are termed bins.: "Chromosomes are divided into series of consecutive fragments of the same size, which are termed bins."
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well.: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
- [readme] bin/produceTSSmaps  -w 1000 tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_2 5000: "bin/produceTSSmaps  -w 1000 tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_2 5000"
