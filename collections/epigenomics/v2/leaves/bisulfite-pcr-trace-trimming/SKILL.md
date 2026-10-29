---
name: bisulfite-pcr-trace-trimming
description: Use when raw .ab1 Sanger trace files from bisulfite sequencing PCR show
  low-quality ends or mixed/overlapping base peaks, and a trimmed sequence must be
  produced before computing per-CpG methylation percentages.
when_to_use_negative:
- The analysis goal is computing CpG methylation percentages directly from already-trimmed,
  high-quality reads — trimming is only a precursor step in the ABSP workflow
- Input data is not Sanger chromatogram (.ab1) trace data from BSP (e.g. NGS reads)
  — ABSP is built for trace files as input
- The sequence ends are already known to be clean and the trimming plot shows no low-quality
  or mixed-base ends; aggressive trimming risks losing covered CpG sites
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: ABSP
  role: R tool that combines Phred quality-score trimming and primary peak-ratio (mixed
    base) trimming to produce the final trimmed sequence before methylation computation
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP and its trimming routines (ABSP_individual_analysis.Rmd
    parameters th_quality_error, th_mixed_position, th_mixed_perc) are executed
  repo: https://www.r-project.org/
- name: sangeranalyseR
  role: Used within R to parse .ab1 base calls with per-position Phred quality scores
    and primary/mixed peak ratios feeding the two trimming methods
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:0f78515cb954a5d85fd4cc6abe26f991feb0b3ae9c91284d7b36c0d06d2c05f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - sangeranalyseR
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bisulfite-pcr-trace-trimming

## Summary

Trim raw Sanger .ab1 chromatogram reads before CpG methylation computation by combining Phred quality-score-based trimming with primary-peak-ratio-based (mixed base) trimming, as implemented in the ABSP R tool for bisulfite sequencing PCR (BSP) data.

## When to use

Use when raw .ab1 Sanger trace files from bisulfite sequencing PCR show low-quality ends or mixed/overlapping base peaks, and a trimmed sequence must be produced before computing per-CpG methylation percentages.

## When NOT to use

- The analysis goal is computing CpG methylation percentages directly from already-trimmed, high-quality reads — trimming is only a precursor step in the ABSP workflow
- Input data is not Sanger chromatogram (.ab1) trace data from BSP (e.g. NGS reads) — ABSP is built for trace files as input
- The sequence ends are already known to be clean and the trimming plot shows no low-quality or mixed-base ends; aggressive trimming risks losing covered CpG sites

## Inputs

- Raw Sanger sequencing trace files (.ab1) from bisulfite sequencing PCR
- ABSP trimming parameters: th_quality_error, th_mixed_position, th_mixed_perc

## Outputs

- Final trimmed sequence per read (e.g. written to trimmed_sequences.fasta)
- Trimming plot showing quality-score (orange), primary-ratio (blue), and combined final (green) trimming boundaries for QC

## How to apply

Parse each .ab1 trace with per-position Phred quality scores and primary/mixed peak ratios (e.g. via sangeranalyseR in R). Apply quality-score trimming with th_quality_error = 0.001 (base-calling accuracy 99.9%, Phred Q30) to flag low-quality ends, and primary peak-ratio trimming with th_mixed_position = 0.75 (minimum primary peak ratio for a position to be non-mixed) and th_mixed_perc = 75% (minimum percentage of non-mixed positions), scanning windows of n = 3 to 15 consecutive non-mixed positions. Obtain the final trimmed sequence by overlapping the results of both trimming methods, so ends are neither trimmed too short nor left low-quality; this combined sequence feeds the subsequent CpG methylation computation. If the final sequence is too short, inspect the trimming plot and relax th_quality_error (e.g. to 0.01, i.e. Q20) or adjust th_mixed_position, editing these thresholds in ABSP_individual_analysis.Rmd (lines 607, 612, 614).

## Related tools

- **ABSP** (R tool that combines Phred quality-score trimming and primary peak-ratio (mixed base) trimming to produce the final trimmed sequence before methylation computation) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP and its trimming routines (ABSP_individual_analysis.Rmd parameters th_quality_error, th_mixed_position, th_mixed_perc) are executed) — https://www.r-project.org/
- **sangeranalyseR** (Used within R to parse .ab1 base calls with per-position Phred quality scores and primary/mixed peak ratios feeding the two trimming methods)

## Examples

```
Rscript -e 'th_quality_error <- 0.001; th_mixed_position <- 0.75; th_mixed_perc <- 0.75; source("app.R")'
```

## Evaluation signals

- The trimming plot shows three boundaries: quality-score-based (orange), primary-ratio-based (blue), and combined final (green); the final trimmed sequence corresponds to the overlap of the two methods
- The final trimmed sequence is neither too short nor retaining low-quality ends; if too short or if 'no CpG sites were found covered by sequencing results' is raised, thresholds were too aggressive
- With default parameters, th_quality_error = 0.001 (99.9% base-calling accuracy, Phred Q30), th_mixed_position = 0.75, th_mixed_perc = 75% are respected in the ABSP_individual_analysis.Rmd script
- Downstream methylation analysis completes: the trimmed reads cover CpG sites and produce a meth_table with values within 0-100%
- Trimming results reproduce the ABSP example outputs for the deposited example .ab1 reads (e.g. the CDH1 example data)

## Limitations

- Trimming too aggressively can yield a very short final sequence covering only a few CpG sites, or abort the analysis with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Thresholds may need manual tuning: if quality-score trimming is too aggressive, lower th_quality_error (e.g. from 0.001 to 0.01, i.e. Q30 to Q20); if primary-ratio trimming is too strict, adjust th_mixed_position
- Trimming parameters are edited directly in the ABSP_individual_analysis.Rmd script (lines 607, 612, 614), not exposed as simple app inputs

## Evidence

- [methods] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [methods] By default, this is set to 0.001: "the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001"
- [methods] this is the minimum primary peak ratio for a position to be considered non-mixed: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed"
- [readme] The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green).: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)."
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
