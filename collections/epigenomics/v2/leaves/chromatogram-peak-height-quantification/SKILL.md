---
name: chromatogram-peak-height-quantification
description: Use when you have .ab1 Sanger trace files plus a reference .fasta from
  a bisulfite sequencing PCR experiment and need per-CpG methylation percentages computed
  from dye peak heights.
when_to_use_negative:
- Your data are high-throughput reads (FASTQ/BAM) rather than Sanger .ab1 chromatogram
  traces — ABSP uses raw .ab1 trace files as input
- You need genome-wide methylation rather than region-specific CpG methylation of
  one BSP-amplified DNA region
- A reference genome is unavailable and you cannot accept arbitrary coordinates —
  while methylation computation still works, genomic plot coordinates and CpG position
  labels become meaningless and should be set to 'CpG numbers' or 'None'
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3676
tools:
- name: ABSP
  role: R tool that parses .ab1 traces, trims reads, aligns to the reference, and
    computes CpG methylation percentages from dye peak heights
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP is installed and executed (install R and
    RStudio, run app.R)
- name: Shiny
  role: Built-in R Shiny app providing the user interface (Individual, Grouped, Multiple
    analyses tabs)
- name: Gviz
  role: Used for genomic visualization plots of the reference sequence when a BSgenome/reference
    genome is provided
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:d045d24d1bddec8c2c6cf1fbf9dc6fb5c63a8e2308fa795b28ac3c52867bcb33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - Gviz
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# chromatogram-peak-height-quantification

## Summary

Quantify per-CpG methylation percentages directly from Sanger chromatogram (.ab1) dye peak heights, as implemented in the ABSP R tool for bisulfite sequencing PCR (BSP) data, avoiding error-prone manual trace reading.

## When to use

Use when you have .ab1 Sanger trace files plus a reference .fasta from a bisulfite sequencing PCR experiment and need per-CpG methylation percentages computed from dye peak heights.

## When NOT to use

- Your data are high-throughput reads (FASTQ/BAM) rather than Sanger .ab1 chromatogram traces — ABSP uses raw .ab1 trace files as input
- You need genome-wide methylation rather than region-specific CpG methylation of one BSP-amplified DNA region
- A reference genome is unavailable and you cannot accept arbitrary coordinates — while methylation computation still works, genomic plot coordinates and CpG position labels become meaningless and should be set to 'CpG numbers' or 'None'

## Inputs

- Sanger sequencing trace files in .ab1 format (direct-BSP or cloning-BSP)
- Reference sequence in .fasta format (with coordinates in the FASTA header, e.g. 'chr1:1-200' for custom sequences)
- Optional reference genome assembly for genomic plots (e.g. GCF_900626175.2_cs10 genomic.fna.gz)

## Outputs

- Per-CpG methylation percentage table per sample (meth_table, e.g. 'CDH1_High methylated DNA_3_meth_table.png') in the results folder
- .html analysis report in the reports folder
- Per-read bisulfite conversion rates with QC flags relative to the 0.9 threshold
- Methylation profile and lollipop plots when samples are grouped

## How to apply

Load the .ab1 trace files and the reference .fasta into ABSP (R) and extract primary peak heights per position. Trim low-quality read ends using ABSP's combined trimming: Phred quality-score-based trimming with th_quality_error = 0.001 (99.9% base-calling accuracy, Phred 30) and primary-peak-ratio-based trimming with th_mixed_position = 0.75 and th_mixed_perc = 75%. Align the trimmed read to the reference to locate covered CpG sites, then compute each CpG methylation percentage as peakC/(peakC+peakT)×100 for forward reads and peakG/(peakG+peakA)×100 for reverse reads. Optionally validate bisulfite conversion by averaging non-CpG conversion ratios (peakT/(peakC+peakT) forward, peakA/(peakG+peakA) reverse) per read against the default 0.9 threshold. Write the per-CpG methylation table (e.g. 'CDH1_High methylated DNA_3_meth_table.png') to the results folder and review the generated .html report.

## Related tools

- **ABSP** (R tool that parses .ab1 traces, trims reads, aligns to the reference, and computes CpG methylation percentages from dye peak heights) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP is installed and executed (install R and RStudio, run app.R))
- **Shiny** (Built-in R Shiny app providing the user interface (Individual, Grouped, Multiple analyses tabs))
- **Gviz** (Used for genomic visualization plots of the reference sequence when a BSgenome/reference genome is provided)

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # then use the 'Individual analysis' tab with CDH1 .ab1 traces + .fasta reference
```

## Evaluation signals

- The meth_table output file exists in the results folder and contains one methylation percentage per covered CpG site, all within 0–100
- Forward-read CpG values equal peakC/(peakC+peakT)×100 and reverse-read values equal peakG/(peakG+peakA)×100 from the chromatogram peak heights
- Each per-read bisulfite conversion rate is numeric in [0,1]; reads at or above the default 0.9 threshold are labeled 'pass'
- Results reproduce the deposited ABSP example output for the CDH1 high methylated DNA sample
- Trimming plot shows the combined quality-based (orange) and primary-ratio (blue) trimming boundaries merged into the final trimmed sequence (green) without over-trimming

## Limitations

- Reference genome is only needed for genomic plot visualization; without it, coordinates in tables/plots are arbitrary and must not be considered
- Over-aggressive trimming can leave a very short sequence with few covered CpG sites, or abort the analysis with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results'; thresholds th_quality_error (default 0.001, line 607), th_mixed_position (default 0.75, line 612) and th_mixed_perc (default 75%, line 614) in ABSP_individual_analysis.Rmd may need adjustment
- On Linux, the initial package installation during first launch may take considerable time

## Evidence

- [other] ABSP computes each CpG methylation percentage from dye-signal peak heights as peakC/(peakC+peakT)×100 for forward sequencing reads and peakG/(peakG+peakA)×100 for reverse sequencing reads.: [evidence span withheld — anchor status: not_found]
- [other] ABSP computes the bisulfite conversion rate for each cytosine position outside CpG sites, and the average rate on the sequence must be higher than a provided threshold, which is 0.9 by default.: [evidence span withheld — anchor status: not_found]
- [readme] the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30.: "By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30."
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [methods] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format.: "add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
