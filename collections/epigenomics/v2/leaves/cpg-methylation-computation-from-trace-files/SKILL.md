---
name: cpg-methylation-computation-from-trace-files
description: Use when input data are raw .ab1 chromatogram trace files from bisulfite
  sequencing PCR (direct-BSP or cloning-BSP) and the goal is to estimate and compare
  CpG methylation percentages of a studied DNA region without manual chromatogram
  inspection.
when_to_use_negative:
- Input is already trimmed, aligned short-read bisulfite sequencing data (e.g. FASTQ/BAM)
  — ABSP is designed for raw .ab1 chromatogram traces
- The analysis goal is genome-wide methylation profiling — ABSP targets region-specific
  CpG methylation from BSP experiments
- Only methylation computation is needed and no reference genome is available but
  genomic coordinate display is required — arbitrary genomes distort displayed coordinates
  (use 'CpG numbers' or 'None' labels and ignore coordinates)
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: ABSP
  role: Automated R tool that parses .ab1 trace files, trims reads, computes CpG methylation
    percentages, and generates plots and comparative statistics
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP is implemented and executed
- name: RStudio
  role: IDE used to open the ABSP R project and launch the app via the 'Run App' button
- name: shiny
  role: Built-in user interface of ABSP (Individual analysis, Grouped analysis, Multiple
    analyses tabs)
- name: Gviz
  role: Renders genomic visualization plots of the reference sequence; works best
    with a BSgenome object
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:6d7a5eecb483ebc304cc5c4d04edd3e440e14077668d01d57311db6ee1c1c0b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - Gviz
  - R
  - RStudio
  - shiny
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cpg-methylation-computation-from-trace-files

## Summary

Computes region-specific CpG methylation percentages directly from Sanger sequencing trace files (.ab1) using ABSP, an automated R/Shiny workflow that spans trimming, per-CpG methylation computation, visualization, and comparative statistics for direct-BSP and cloning-BSP data.

## When to use

Use when input data are raw .ab1 chromatogram trace files from bisulfite sequencing PCR (direct-BSP or cloning-BSP) and the goal is to estimate and compare CpG methylation percentages of a studied DNA region without manual chromatogram inspection.

## When NOT to use

- Input is already trimmed, aligned short-read bisulfite sequencing data (e.g. FASTQ/BAM) — ABSP is designed for raw .ab1 chromatogram traces
- The analysis goal is genome-wide methylation profiling — ABSP targets region-specific CpG methylation from BSP experiments
- Only methylation computation is needed and no reference genome is available but genomic coordinate display is required — arbitrary genomes distort displayed coordinates (use 'CpG numbers' or 'None' labels and ignore coordinates)

## Inputs

- Sequencing trace files in .ab1 format (direct-BSP or cloning-BSP)
- Reference sequence of the studied region in .fasta format (with coordinates, e.g. 'chr1:1-200' for custom sequences)
- Optional reference genome assembly (e.g. GCF_900626175.2_cs10 .fna.gz) for genomic plots only
- Input entry tables for the 'Multiple analyses' tab when batching samples

## Outputs

- Per-CpG methylation tables (e.g. CDH1_High methylated DNA_3_meth_table.png)
- Lollipop plots and methylation profile plots from grouped analysis
- Trimming plot showing quality-score (orange), primary-ratio (blue), and final trimmed (green) sequence boundaries
- .html analysis report in the 'reports' folder
- Result files in the 'results' folder

## How to apply

Install R and RStudio, open 'ABSP Rproject.Rproject', and launch the Shiny app from 'app.R' (clicking 'Yes' if prompted to install the shiny package). In the 'Individual analysis' tab, supply the reference sequence in .fasta format and the .ab1 sequencing files, then run trimming that combines the Phred quality-score method (th_quality_error = 0.001, i.e. 99.9% base-calling accuracy, line 607 of ABSP_individual_analysis.Rmd) with the primary peak-ratio method (th_mixed_position = 0.75 at line 612; th_mixed_perc = 75% at line 614). Inspect the 'Trimming plot' (orange = quality-score method, blue = primary-ratio method, green = final trimmed sequence) to confirm trimming did not remove low-quality ends too aggressively or leave them in place. Then run the analysis to compute per-CpG methylation levels from primary peak heights, review the .html report in the 'reports' folder, and gather results via the 'Grouped analysis' tab for lollipop plots, methylation profile plots, and comparative statistics. Inputs (.fasta reference and .ab1 traces) come from the ABSP 'examples' folder for validation, and the reference genome is optional — needed only for genomic visualization, not methylation computation.

## Related tools

- **ABSP** (Automated R tool that parses .ab1 trace files, trims reads, computes CpG methylation percentages, and generates plots and comparative statistics) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP is implemented and executed)
- **RStudio** (IDE used to open the ABSP R project and launch the app via the 'Run App' button)
- **shiny** (Built-in user interface of ABSP (Individual analysis, Grouped analysis, Multiple analyses tabs))
- **Gviz** (Renders genomic visualization plots of the reference sequence; works best with a BSgenome object)

## Examples

```
Open 'ABSP Rproject.Rproject' in RStudio, open 'app.R', and click 'Run App'; then in the Individual analysis tab load the CDH1 .fasta reference and EpigenDx .ab1 files and run the analysis.
```

## Evaluation signals

- Reproduced example metrics match reported values: CpG site #4 methylation 0.47% ± 0.81% (low-methylated) vs 81.44% ± 3.96% (high-methylated) with P = 0.00051; mean methylation 1.43% ± 1.87% vs 95.54% ± 0.67% with P = 2.34e−05
- Each computed CpG methylation percentage is a valid proportion between 0 and 1 (or 0–100%)
- Trimming plot shows the three boundaries: quality-score method (orange), primary-ratio method (blue), and final trimmed sequence (green), with the trimmed sequence neither too short nor retaining low-quality ends
- QC outcome consistent with example data (e.g. 5/6 forward reads fail trimming/QC for the CDH1 direct-BSP dataset)
- A .html report is generated in the 'reports' folder and output files appear in the 'results' folder after each analysis

## Limitations

- Manual analysis of chromatogram traces for direct-BSP is repetitive and prone to error — the automation exists to avoid this, but parameter tuning may be needed
- If the Phred quality-score trimming is too aggressive (sequence too short), lower th_quality_error (e.g. from 0.001 to 0.01, Phred 30 → 20); if primary peak-ratio trimming is too strict, adjust th_mixed_position and th_mixed_perc
- Analysis aborts with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results' when trimming leaves too few CpG sites
- Reference genome is only needed for genomic visualization; using an arbitrary genome makes displayed coordinates meaningless and they must not be considered in tables or plots

## Evidence

- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [methods] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
- [readme] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [readme] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
