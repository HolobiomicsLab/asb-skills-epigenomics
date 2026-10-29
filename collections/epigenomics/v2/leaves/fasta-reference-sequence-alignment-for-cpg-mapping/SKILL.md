---
name: fasta-reference-sequence-alignment-for-cpg-mapping
description: Use when analyzing direct-BSP or cloning-BSP data from raw Sanger .ab1
  trace files that must be trimmed, matched to a .fasta reference sequence, and converted
  into per-CpG methylation percentages for one or more sample groups.
when_to_use_negative:
- The input is next-generation bisulfite sequencing data (e.g. FASTQ amplicon reads)
  rather than Sanger .ab1 chromatogram traces, which is what ABSP consumes
- The goal is genome-wide methylation profiling rather than region-specific CpG analysis
  of a PCR-amplified locus
- The reference sequence is absent from any reference genome and only methylation
  percentages are needed — the genome itself is unnecessary for the computation, so
  no genome assembly step is required
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: ABSP
  role: Automated R tool that trims .ab1 traces, aligns them to the .fasta reference,
    computes CpG methylation percentages, and produces plots and reports
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP is installed and executed (with RStudio
    as the IDE)
- name: shiny
  role: Provides the ABSP user interface ('Individual analysis', 'Grouped analysis',
    'Multiple analyses' tabs) launched from app.R
- name: Gviz
  role: Renders genomic visualization plots of the reference sequence; works best
    with a BSgenome object
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:0fc8d09c67c711a0746e5d5751d029552adb6ee2d679c4b4e82a3a5c20379882
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# FASTA reference sequence alignment for CpG mapping

## Summary

Skill for the ABSP R tool's core step of aligning trimmed Sanger bisulfite sequencing PCR (.ab1) chromatogram reads against a region-specific reference sequence in .fasta format to compute per-CpG methylation percentages, with optional genomic visualization via Gviz.

## When to use

Use when analyzing direct-BSP or cloning-BSP data from raw Sanger .ab1 trace files that must be trimmed, matched to a .fasta reference sequence, and converted into per-CpG methylation percentages for one or more sample groups.

## When NOT to use

- The input is next-generation bisulfite sequencing data (e.g. FASTQ amplicon reads) rather than Sanger .ab1 chromatogram traces, which is what ABSP consumes
- The goal is genome-wide methylation profiling rather than region-specific CpG analysis of a PCR-amplified locus
- The reference sequence is absent from any reference genome and only methylation percentages are needed — the genome itself is unnecessary for the computation, so no genome assembly step is required

## Inputs

- Reference sequence of the studied region in .fasta format (header may carry coordinates such as 'chr1:1-200' for custom reporter sequences)
- Raw Sanger sequencing trace files in .ab1 format from bisulfite PCR (direct-BSP or cloning-BSP)
- Optional reference genome (e.g. the GCF_900626175.2_cs10 assembly or a BSgenome) for genomic visualization plots
- Optional input tables for the 'Multiple analyses' tab when batching samples or groupings

## Outputs

- Per-CpG methylation tables (e.g. CDH1_High methylated DNA_3_meth_table.png) in the 'results' folder
- Lollipop plots and methylation profile plots from the grouped analysis
- Comparative statistics between sample groups (e.g. P = 0.00051 for CpG site #4, low vs high methylated CDH1 DNA)
- An .html analysis report per individual and grouped analysis in the 'reports' folder

## How to apply

Install R and RStudio, open 'ABSP Rproject.Rproject' and launch the ABSP Shiny app from 'app.R', clicking 'Yes' if prompted to install the shiny package. In the 'Individual analysis' tab, supply the reference sequence in .fasta format together with the .ab1 sequencing files, then run the analysis to compute CpG methylation levels. Each read is trimmed by combining the Phred quality-score method (default th_quality_error = 0.001, i.e. 99.9% base-calling accuracy / Phred 30, line 607 of ABSP_individual_analysis.Rmd) with the primary peak-ratio method (th_mixed_position = 0.75 at line 612, th_mixed_perc = 75% at line 614); inspect the 'Trimming plot' where orange marks the quality-score trim, blue the primary-ratio trim, and green the final trimmed sequence. The reference genome is optional and only needed for genomic-coordinate plots (Gviz works best with a BSgenome); an arbitrary genome or 'chr1:1-XXX' coordinates can be used without affecting methylation computation. Gather individually analyzed samples in the 'Grouped analysis' tab to generate lollipop and methylation profile plots and comparative statistics between groups, then collect methylation tables, plots, and the .html report from the 'results' and 'reports' folders.

## Related tools

- **ABSP** (Automated R tool that trims .ab1 traces, aligns them to the .fasta reference, computes CpG methylation percentages, and produces plots and reports) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP is installed and executed (with RStudio as the IDE))
- **shiny** (Provides the ABSP user interface ('Individual analysis', 'Grouped analysis', 'Multiple analyses' tabs) launched from app.R)
- **Gviz** (Renders genomic visualization plots of the reference sequence; works best with a BSgenome object)

## Examples

```
shiny::runApp("app.R")  # launch the ABSP Shiny app, then use the 'Individual analysis' tab with the CDH1 .fasta reference and .ab1 trace files
```

## Evaluation signals

- On the EpigenDx CDH1 example (80-8061-HGHM5 high- vs 80-8062-HGUM5 low-methylated DNA), reproduce CpG site #4 methylation of 81.44% (±3.96%) vs 0.47% (±0.81%) with P = 0.00051
- Reproduce mean methylation 95.54% (±0.67%) vs 1.43% (±1.87%) with P = 2.34e−05 between the high- and low-methylated control DNAs
- Confirm the expected QC outcome that 5/6 forward reads fail trimming/QC on the example data
- Check the 'Trimming plot': the final trimmed sequence (green) should be the combination of the quality-score (orange) and primary-ratio (blue) trims and should not leave low-quality ends
- Verify that output .html reports are generated in the 'reports' folder and methylation tables/plots in the 'results' folder for each analysis

## Limitations

- Sequences may become too short after trimming, or the analysis aborts with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results'; adjust th_quality_error (e.g. to 0.01 for Phred 20), th_mixed_position, or th_mixed_perc
- The reference genome is only needed for visualization; using an arbitrary genome makes displayed coordinates meaningless, so 'CpG numbers' or 'None' CpG label options should be chosen in Grouped Analysis
- On Linux, the first-launch package installation may take a considerable amount of time
- Custom reporter sequences require arbitrary FASTA header coordinates (e.g. 'chr1:1-200' for a 200 bp sequence)

## Evidence

- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format.: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format."
- [readme] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [readme] the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages: "the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages"
- [methods] For genomic plots, the Gviz package works best with a [BSgenome]: "For genomic plots, the Gviz package works best with a [BSgenome]"
