---
name: forward-reverse-strand-methylation-parsing
description: Use when analyzing direct-BSP .ab1 Sanger trace files to quantify CpG
  methylation at a specific region, and per-site methylation percentages must be derived
  from dye-signal peak heights at forward (C/T) or reverse (G/A) positions.
when_to_use_negative:
- Input data are already aligned high-throughput reads (FASTQ/BAM); this skill parses
  Sanger chromatogram peak heights from .ab1 files only
- The goal is genome-wide methylation profiling; ABSP is designed for region-specific
  CpG methylation from BSP experiments
- Only plotting or statistical comparison between sample groups is needed without
  per-CpG peak-height computation; use the grouped-analysis features instead
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ABSP
  role: Automated R tool that computes per-CpG methylation percentages from .ab1 chromatogram
    peak heights using the forward peakC/(peakC+peakT) and reverse peakG/(peakG+peakA)
    formulas
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP is installed and executed for the methylation
    computation
- name: Shiny
  role: Built-in user interface (app.R) through which the individual sample analysis
    and methylation table computation are launched
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:1c0e659a920df8d48f51bf4c4e538c61049d60b3b2e9faacc938250ba2c74363
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# forward-reverse-strand-methylation-parsing

## Summary

Computes per-CpG methylation percentages from Sanger bisulfite sequencing PCR (BSP) chromatogram peak heights, using peakC/(peakC+peakT)×100 for forward reads and peakG/(peakG+peakA)×100 for reverse reads, as implemented in the ABSP R tool.

## When to use

Use when analyzing direct-BSP .ab1 Sanger trace files to quantify CpG methylation at a specific region, and per-site methylation percentages must be derived from dye-signal peak heights at forward (C/T) or reverse (G/A) positions.

## When NOT to use

- Input data are already aligned high-throughput reads (FASTQ/BAM); this skill parses Sanger chromatogram peak heights from .ab1 files only
- The goal is genome-wide methylation profiling; ABSP is designed for region-specific CpG methylation from BSP experiments
- Only plotting or statistical comparison between sample groups is needed without per-CpG peak-height computation; use the grouped-analysis features instead

## Inputs

- Raw Sanger sequencing trace files (.ab1) from direct bisulfite sequencing PCR
- Reference sequence in .fasta format (with coordinates in the FASTA header, e.g. chr1:1-200)
- Optional: reference genome assembly (e.g. GCF_900626175.2_cs10_genomic.fna.gz) for genomic visualization only

## Outputs

- Per-CpG methylation percentage table (one value in 0-100 per covered CpG site) in the results folder
- Methylation table image (e.g. <sample>_meth_table.png)
- HTML analysis report in the reports folder

## How to apply

Load the deposited .ab1 sequencing traces together with the reference sequence in .fasta format into ABSP. First trim low-quality ends with ABSP's combined trimming: Phred quality-score-based trimming with th_quality_error = 0.001 (base-calling accuracy 99.9%, Q30) and primary-peak-ratio-based trimming with th_mixed_position = 0.75 and th_mixed_perc = 75%. Align the trimmed sequences to the reference to locate the CpG positions covered by each read. For each covered CpG, compute the methylation percentage from the chromatogram dye peak heights as peakC/(peakC+peakT)×100 on forward sequencing reads and peakG/(peakG+peakA)×100 on reverse sequencing reads. Write the per-CpG methylation level table to the results folder and render it as the methylation table image (e.g. 'CDH1_High methylated DNA_3_meth_table.png'). Note the reference genome is only needed for visualization in genomic plots, not for computing the methylation percentages.

## Related tools

- **ABSP** (Automated R tool that computes per-CpG methylation percentages from .ab1 chromatogram peak heights using the forward peakC/(peakC+peakT) and reverse peakG/(peakG+peakA) formulas) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP is installed and executed for the methylation computation)
- **Shiny** (Built-in user interface (app.R) through which the individual sample analysis and methylation table computation are launched)

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # then use the 'Individual analysis' tab with .ab1 traces and a .fasta reference to run the methylation computation
```

## Evaluation signals

- The meth_table output file exists in the results folder for each analyzed sample
- The table contains one methylation percentage per covered CpG site, and all values fall within 0-100
- The output reproduces the deposited ABSP example result for the CDH1 high methylated DNA sample (CDH1_High methylated DNA_3_meth_table.png)
- Forward reads yield percentages from peakC/(peakC+peakT) and reverse reads from peakG/(peakG+peakA) — spot-check one covered CpG manually against the chromatogram
- The .html report is generated in the reports folder and the trimming plot shows combined quality-score (orange), primary-ratio (blue) and final (green) trimming

## Limitations

- If trimming is too aggressive, the trimmed sequence can become too short with only a few covered CpG sites, or the analysis stops with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'; th_quality_error (default 0.001, line 607 of ABSP_individual_analysis.Rmd) may be lowered to 0.01 (Q20) and th_mixed_position (default 0.75) / th_mixed_perc (default 75%) adjusted
- A reference genome is not required for computing methylation percentages; with an arbitrary genome, coordinates in tables/plots must not be taken into consideration and 'CpG numbers' or 'None' CpG position labels are recommended
- Manual chromatogram analysis of direct-BSP results is repetitive and error-prone, which is the motivation for the automated ABSP workflow

## Evidence

- [other] ABSP computes each CpG methylation percentage from dye-signal peak heights as peakC/(peakC+peakT)×100 for forward sequencing reads and peakG/(peakG+peakA)×100 for reverse sequencing reads.: [evidence span withheld — anchor status: not_found]
- [readme] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [methods] the reference genome is only necessary for visualization of the reference sequence in genomic plots: "the reference genome is only necessary for visualization of the reference sequence in genomic plots"
- [readme] this is the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001: "the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001"
