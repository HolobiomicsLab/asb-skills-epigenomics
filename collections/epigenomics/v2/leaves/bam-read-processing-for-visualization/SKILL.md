---
name: bam-read-processing-for-visualization
description: Use when raw ChIP-seq or RNA-seq data must be mapped, cleaned, and packaged
  with chromosome/refGene annotation files so SeqCode commands like buildChIPprofile,
  produceTSSplots, or produceTSSmaps can generate plots and heatmaps.
when_to_use_negative:
- The goal is generating genome-wide profiles in bigwig format — the paper defers
  this to future versions of SeqCode
- The task is upstream peak calling or quality control — use MACS or dedicated QC
  tools and feed the resulting processed data to SeqCode
- The web server is the only option available — it is a teaching resource with file-size
  limits; the command line release has no restrictions
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: SeqCode
  role: Consumes the cleaned BAM with chromosome and RefSeq annotation files to produce
    meta-plots, heatmaps and other graphics via embedded R scripts
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Maps the raw reads over the chosen genome assembly (mouse mm9/mm10, human
    hg38, fruit fly dm3) to create the BAM input
- name: SAMtools
  role: Source code integrated into SeqCode (including BCFtools and HTSlib) to efficiently
    read BAM/SAM files; can also be used to exclude unaligned reads upstream
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:ea461dcf5658253bc00d2a48ca5be0a323752d390e33f64804aee4eea05a2619
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# bam-read-processing-for-visualization

## Summary

Prepares mapped sequencing reads (BAM/SAM) and genome annotation inputs for high-quality graphical analysis (meta-plots, heatmaps) with SeqCode, ensuring unaligned reads are removed and the correct UCSC chromosome-size and RefSeq refGene files accompany each assembly.

## When to use

Use when raw ChIP-seq or RNA-seq data must be mapped, cleaned, and packaged with chromosome/refGene annotation files so SeqCode commands like buildChIPprofile, produceTSSplots, or produceTSSmaps can generate plots and heatmaps.

## When NOT to use

- The goal is generating genome-wide profiles in bigwig format — the paper defers this to future versions of SeqCode
- The task is upstream peak calling or quality control — use MACS or dedicated QC tools and feed the resulting processed data to SeqCode
- The web server is the only option available — it is a teaching resource with file-size limits; the command line release has no restrictions

## Inputs

- Raw sequencing data (e.g. SRA accession SRX367147) retrieved from NCBI GEO
- Input BAM or SAM file of mapped reads (unaligned reads excluded)
- UCSC chromosome sizes file for the assembly (e.g. ChromInfo.txt)
- RefSeq refGene.txt transcript annotation file for the assembly
- Gene list file (e.g. genes_chr10.txt) for TSS-centered plots/maps

## Outputs

- SeqCode output folder with averaged TSS meta-plot PDFs (e.g. PlotHEATmap_test_1_5000.pdf)
- Heatmaps of sequencing signal around TSS/TES/peak centers
- Data files and generated R scripts (Rscript_produceTSSmaps.txt) stored alongside results

## How to apply

Retrieve raw data for the experiment (e.g. SRX367147 mESC H3K4me3 ChIP-seq from NCBI GEO) and map reads with Bowtie to the appropriate assembly (mm9, mm10, hg38, dm3). Exclude unaligned reads from the resulting BAM, since SeqCode commands require aligned-read BAM files. For the target assembly, obtain two UCSC files required by most commands: the chromosome list with sizes (chromInfo.txt) and the RefSeq transcript annotations (refGene.txt), plus a gene list if running TSS-centered tools. SeqCode internally integrates the SamTools and HTS C libraries to read BAM and SAM files, so no external SAMtools run is needed by SeqCode itself, though SAMtools may be used upstream. Run the binary (e.g. bin/produceTSSplots or bin/produceTSSmaps) with chromosome file, refGene file, BAM, and gene list over a window (default 5000 bp around the TSS), then execute the emitted R script (R CMD BATCH Rscript_produceTSSmaps.txt) to render final PDFs.

## Related tools

- **SeqCode** (Consumes the cleaned BAM with chromosome and RefSeq annotation files to produce meta-plots, heatmaps and other graphics via embedded R scripts) — https://github.com/eblancoga/seqcode
- **Bowtie** (Maps the raw reads over the chosen genome assembly (mouse mm9/mm10, human hg38, fruit fly dm3) to create the BAM input)
- **SAMtools** (Source code integrated into SeqCode (including BCFtools and HTSlib) to efficiently read BAM/SAM files; can also be used to exclude unaligned reads upstream) — https://github.com/eblancoga/seqcode

## Examples

```
bin/produceTSSmaps -w 1000 tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_2 5000; R CMD BATCH tests/inputs/Rscript_produceTSSmaps.txt
```

## Evaluation signals

- The BAM given to SeqCode commands contains no unaligned reads (SeqCode commands require aligned reads be retained)
- A PlotHEATmap PDF is produced for every window setting tested (e.g. PlotHEATmap_test_1_5000.pdf and PlotHEATmap_test_2_5000.pdf at -w 1000)
- The R script stored in each output folder executes successfully with R CMD BATCH to render final PDFs
- Signal patterns in TSS heatmaps/meta-plots remain biologically consistent across different -w window resolutions, with only graphical quality changing at coarser windows
- The verbose flag (-v) reports each pipeline stage completing with [DONE] and 'Successful termination'

## Limitations

- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions of SeqCode
- Spike-in normalization factor must be manually introduced by users
- When compiling SAMtools libraries, pthread warning messages may appear but can be safely skipped
- Unaligned reads must be explicitly excluded from BAM files before SeqCode commands can process them

## Evidence

- [methods] Exclusion of unaligned reads: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] Read mapping with Bowtie: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] SAMtools integration: "Our programs internally integrate the SamTools and HTS C libraries18 in order to efficiently read BAM and SAM files."
- [readme] Required genome annotation files: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] R script execution for plots: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
