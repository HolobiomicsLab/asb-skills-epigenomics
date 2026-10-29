---
name: window-resolution-parameter-tuning
description: Use when running SeqCode visualization commands on BAM/SAM read data
  and you must choose or justify a window resolution for TSS/peak meta-plots and heatmaps,
  or verify results are robust to coarser binning.
when_to_use_negative:
- The question is about quantitative read counts or normalization values (e.g. recoverChIPlevels
  output), where bin size directly affects the computed values, not just graphics
- You need genome-wide profiles in bigwig format — the article states this is not
  supported and deferred to future versions
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Provides produceTSSmaps/produceTSSplots/buildChIPprofile with the -w window
    resolution option; internally generates plots via stored R scripts
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Integrated (SAMtools/HTSlib) inside SeqCode to read the BAM/SAM mapped-read
    inputs used at each window setting
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:3cebfd55e7efb9b79030f85d8a8dd3bd73fa89c54102eb6b2d66b049663c97cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# window-resolution-parameter-tuning

## Summary

Method for assessing whether changing the genome binning window resolution (-w) in SeqCode plotting commands (e.g. produceTSSmaps, produceTSSplots, buildChIPprofile) alters biological conclusions versus only graphical quality, memory needs, and smoothness of meta-plots and heatmaps.

## When to use

Use when running SeqCode visualization commands on BAM/SAM read data and you must choose or justify a window resolution for TSS/peak meta-plots and heatmaps, or verify results are robust to coarser binning.

## When NOT to use

- The question is about quantitative read counts or normalization values (e.g. recoverChIPlevels output), where bin size directly affects the computed values, not just graphics
- You need genome-wide profiles in bigwig format — the article states this is not supported and deferred to future versions

## Inputs

- UCSC chromosome-size file (e.g. ChromInfo.txt) for the target assembly (mm9, mm10, hg38, dm3)
- RefSeq gene transcript annotation file (refGene.txt)
- BAM or SAM file of mapped sequencing reads (e.g. H3K4me3 mESC ChIP-seq, SRX367147) with unaligned reads excluded
- Target gene list file (e.g. genes_chr10.txt)

## Outputs

- TSS heatmap PlotHEATmap PDFs per window setting (e.g. test_1_TSSmap_5000/PlotHEATmap_test_1_5000.pdf and test_2_TSSmap_5000/PlotHEATmap_test_2_5000.pdf)
- Per-run output folders containing data files and the R script (Rscript_produceTSSmaps.txt) that generates the final PDFs
- Comparison across window resolutions demonstrating consistency of signal patterns (e.g. Supplementary Figure 6-style coarse vs default heatmap comparison)

## How to apply

Binning in SeqCode divides chromosomes into consecutive same-size fragments (bins), and commands like bin/produceTSSmaps accept the -w option to set the window resolution. Run the command once at the default resolution (5000 bp around the TSS in the reference workflow) and re-run it with a coarser window (e.g. -w 1000) using identical inputs: the chromosome-size file (ChromInfo.txt), RefSeq refGene.txt annotations, the BAM of mapped reads (unaligned reads excluded), and the target gene list. Execute the generated R script in each output folder (R CMD BATCH Rscript_produceTSSmaps.txt) to render the PDF heatmaps/meta-plots. Then compare the outputs across resolutions: signal patterns, peak positions, and TSS enrichment should remain biologically consistent, with only lower graphical quality (coarser/less smooth heatmaps) at larger windows, confirming that -w is a graphical/performance parameter rather than a biological one.

## Related tools

- **SeqCode** (Provides produceTSSmaps/produceTSSplots/buildChIPprofile with the -w window resolution option; internally generates plots via stored R scripts) — https://github.com/eblancoga/seqcode
- **SAMtools** (Integrated (SAMtools/HTSlib) inside SeqCode to read the BAM/SAM mapped-read inputs used at each window setting) — https://github.com/eblancoga/seqcode

## Examples

```
bin/produceTSSmaps -w 1000 tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_2 5000
```

## Evaluation signals

- A PlotHEATmap PDF is produced for every window setting tested (default 5000 bp and coarse e.g. -w 1000), as checked in tests/outputs/produceTSSmaps/
- Down-sampled window runs retain the same biological pattern (same enriched genes/TSS signal positions) with only reduced graphical quality at coarser windows
- Both default and lower-resolution runs complete via the recorded Perl test stages (e.g. test_produceTSSmaps.pl Stages 1–2) ending in 'Successful termination: [DONE]'
- The stored R script executes successfully (R CMD BATCH Rscript_produceTSSmaps.txt) in each output folder to render the final PDFs
- Each produceTSSmaps run writes its results into a distinct output folder (default in the working directory, or redirected with -x) containing plots, R scripts, and data files

## Limitations

- Lower-resolution (-w) settings trade graphical quality/smoothness of heatmaps for speed and memory, so coarse windows are unsuitable for publication-grade figures
- The comparison method was validated on windowed TSS meta-representations (TSS maps/plots around a 5000 bp window); behavior for other feature types was not established here
- SeqCode relies on RefSeq reference annotations and requires assembly-matched chromosome and refGene files; mismatches will break the binning

## Evidence

- [methods] Binning via chromosome fragments: "Chromosomes are divided into series of consecutive fragments of the same size, which are termed bins."
- [readme] Lower-resolution produceTSSmaps invocation in test script: "%%%% bin/produceTSSmaps  -w 1000 tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_2 5000"
- [readme] Per-window PDF output check: "%%%% Checking output file 2: tests/outputs/produceTSSmaps/test_2_TSSmap_5000/PlotHEATmap_test_2_5000.pdf	[OK]"
- [methods] BAM/SAM input support via integrated libraries: "Our programs internally integrate the SamTools and HTS C libraries18 in order to efficiently read BAM and SAM files."
