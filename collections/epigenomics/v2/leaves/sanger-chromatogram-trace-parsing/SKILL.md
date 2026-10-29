---
name: sanger-chromatogram-trace-parsing
description: Use when the input consists of raw Sanger trace files (.ab1) from bisulfite
  sequencing PCR (BSP) that must be base-called, quality-trimmed, and converted into
  trimmed sequences before methylation percentage computation.
when_to_use_negative:
- Input is already trimmed FASTA/FASTQ sequence or a methylation table — this skill
  operates only on raw .ab1 chromatogram traces
- Data are short-read (e.g. Illumina) bisulfite reads; ABSP and its .ab1-based trimming
  apply to Sanger BSP trace files, not high-throughput reads
- A reference genome is required for methylation computation — the README states the
  reference genome is only necessary for genomic plot visualization, not for trimming
  or methylation percentages
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: ABSP
  role: R tool whose sequencing-trimming step parses .ab1 traces and combines quality-score
    and primary-peak-ratio trimming to produce the final trimmed sequence
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: sangeranalyseR
  role: R library used to parse .ab1 base calls with per-position Phred quality scores
    and primary/mixed peak ratios feeding the trimming step
- name: R / Shiny
  role: Runtime environment and Shiny interface through which the 'Individual analysis'
    tab ingests .ab1 files and renders the trimming plot
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:1faf573f684c221456ff1f3ce88d76a5dc94d78cbea1d56271993a59e9478a46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - R / Shiny
  - sangeranalyseR
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sanger-chromatogram-trace-parsing

## Summary

Parses raw Sanger sequencing chromatogram files (.ab1) with base calls, per-position Phred quality scores, and primary/mixed peak ratios, then trims low-quality and mixed-base ends to yield a final read for downstream CpG methylation analysis in the ABSP R workflow.

## When to use

Use when the input consists of raw Sanger trace files (.ab1) from bisulfite sequencing PCR (BSP) that must be base-called, quality-trimmed, and converted into trimmed sequences before methylation percentage computation.

## When NOT to use

- Input is already trimmed FASTA/FASTQ sequence or a methylation table — this skill operates only on raw .ab1 chromatogram traces
- Data are short-read (e.g. Illumina) bisulfite reads; ABSP and its .ab1-based trimming apply to Sanger BSP trace files, not high-throughput reads
- A reference genome is required for methylation computation — the README states the reference genome is only necessary for genomic plot visualization, not for trimming or methylation percentages

## Inputs

- Raw Sanger sequencing trace files (.ab1 chromatograms) from direct-BSP or cloning-BSP experiments
- Reference sequence in .fasta format (for subsequent alignment/methylation steps)

## Outputs

- Final trimmed sequence per read (overlapping quality-score and primary-ratio trim boundaries)
- Trimming plot with quality-score (orange), primary-ratio (blue), and combined final (green) boundaries for QC inspection
- trimmed_sequences.fasta per read used for downstream CpG methylation percentage computation

## How to apply

Load each .ab1 chromatogram (e.g. the example reads in the ABSP repository's 'examples' folder) and parse base calls together with per-position Phred quality scores and primary/mixed peak ratios. Apply quality-score trimming with the minimum base-calling error probability threshold th_quality_error = 0.001 (99.9% base-calling accuracy, Phred Q30) to flag low-quality ends. Apply primary peak-ratio trimming with th_mixed_position = 0.75 (minimum primary peak ratio for a non-mixed position) and th_mixed_perc = 75% (minimum percentage of non-mixed positions), scanning windows of n = 3 to 15 consecutive non-mixed positions. The final trimmed sequence is obtained by overlapping the two trimming results, so ends are neither trimmed too short nor left low-quality. Inspect the trimming plot — quality-score boundaries in orange, primary-ratio boundaries in blue, combined final boundaries in green — and loosen th_quality_error (e.g. to 0.01, Q20) or th_mixed_position if trimming is too aggressive.

## Related tools

- **ABSP** (R tool whose sequencing-trimming step parses .ab1 traces and combines quality-score and primary-peak-ratio trimming to produce the final trimmed sequence) — https://github.com/ABSP-methylation-tool/ABSP
- **sangeranalyseR** (R library used to parse .ab1 base calls with per-position Phred quality scores and primary/mixed peak ratios feeding the trimming step)
- **R / Shiny** (Runtime environment and Shiny interface through which the 'Individual analysis' tab ingests .ab1 files and renders the trimming plot) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
shiny::runApp("app.R")  # launch ABSP in RStudio, then load .ab1 files in the 'Individual analysis' tab and inspect the 'Sequencing trimming' plot
```

## Evaluation signals

- The trimming plot shows three boundary sets: orange (quality-score), blue (primary-ratio), and a green combined final boundary — the final trimmed sequence must match the overlap of the first two
- Default parameters retained: th_quality_error = 0.001 (Phred Q30), th_mixed_position = 0.75, th_mixed_perc = 75%, scanned over windows of 3–15 consecutive non-mixed positions
- The trimmed sequence must not be so short that the analysis aborts with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Final trimmed sequences are written to trimmed_sequences.fasta and consumed by the subsequent CpG methylation percentage computation without low-quality ends retained

## Limitations

- If the quality-score trimming method is too aggressive, the final sequence may be very short with only a few CpG sites covered; th_quality_error must be lowered (e.g. 0.001 → 0.01, Q30 → Q20)
- If the primary peak-ratio trimming method is too strict, th_mixed_position and th_mixed_perc thresholds must be adjusted manually in ABSP_individual_analysis.Rmd (lines 612 and 614)
- Manual analysis of chromatogram traces for direct-BSP is repetitive and prone to error, which motivated automation — but thresholds still need manual tuning per dataset

## Evidence

- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green).: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)."
- [methods] `th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed"
- [readme] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [methods] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
