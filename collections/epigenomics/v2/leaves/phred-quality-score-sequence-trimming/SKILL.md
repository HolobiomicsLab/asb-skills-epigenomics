---
name: phred-quality-score-sequence-trimming
description: Use when raw Sanger .ab1 bisulfite sequencing PCR (BSP) trace files must
  be quality-trimmed before computing CpG methylation percentages, especially when
  sequences end up too short or low-quality after trimming.
when_to_use_negative:
- Input data are not raw Sanger chromatogram traces (e.g. already-trimmed sequences
  or high-throughput short reads) — this trimming operates on .ab1 files within the
  ABSP BSP pipeline
- The goal is only methylation percentage computation without sequencing trimming
  QC — trimming boundaries are a prerequisite step, not an end in itself
- Reference genome is unavailable — the reference genome is only needed for genomic
  visualization, not for this trimming or for computing methylation percentages
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0622
tools:
- name: ABSP
  role: R tool implementing the quality-score trimming method (th_quality_error) and
    its combination with primary peak-ratio trimming to produce the final trimmed
    sequence
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: sangeranalyseR
  role: Parses .ab1 chromatogram base calls with per-position Phred quality scores
    and primary/mixed peak ratios feeding the trimming
- name: R
  role: Language in which ABSP and the trimming thresholds are implemented and executed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:4ff61c55d7befb2e95c8f0b096fdf034d0124fb20483677a18f8d5c102b35f33
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
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phred-quality-score-sequence-trimming

## Summary

A sequence-trimming step from the ABSP tool that removes low-quality ends of raw .ab1 Sanger chromatogram reads by flagging positions whose base-calling error probability exceeds a Phred-score threshold, then combining the result with primary-peak-ratio trimming to define the final trimmed sequence used for CpG methylation computation.

## When to use

Use when raw Sanger .ab1 bisulfite sequencing PCR (BSP) trace files must be quality-trimmed before computing CpG methylation percentages, especially when sequences end up too short or low-quality after trimming.

## When NOT to use

- Input data are not raw Sanger chromatogram traces (e.g. already-trimmed sequences or high-throughput short reads) — this trimming operates on .ab1 files within the ABSP BSP pipeline
- The goal is only methylation percentage computation without sequencing trimming QC — trimming boundaries are a prerequisite step, not an end in itself
- Reference genome is unavailable — the reference genome is only needed for genomic visualization, not for this trimming or for computing methylation percentages

## Inputs

- Raw Sanger sequencing trace files (.ab1) with base calls and per-position Phred quality scores
- Trimming threshold settings in ABSP_individual_analysis.Rmd (th_quality_error, default 0.001 at line 607)

## Outputs

- Final trimmed sequence per read (overlap of quality-score trimming and mixed-base peak-ratio trimming), used for subsequent CpG methylation percentage computation
- Trimming plot showing quality-score (orange), primary-ratio (blue), and combined final (green) trimming boundaries for QC inspection
- trimmed_sequences.fasta written per read (per source-card workflow)

## How to apply

Parse each .ab1 read with sangeranalyseR inside the ABSP R workflow to obtain per-base Phred quality scores, then apply the quality-score trimming method with the minimum base-calling error probability threshold th_quality_error = 0.001 (default, set at line 607 of ABSP_individual_analysis.Rmd), which corresponds to 99.9% base-calling accuracy (Phred Q30). Because this Phred-based trimming can be overly aggressive on its own, ABSP overlaps it with the primary peak-ratio trimming method (th_mixed_position = 0.75, th_mixed_perc = 75%, windows of n = 3–15 consecutive non-mixed positions) and the overlap of both methods defines the final trimmed sequence used in downstream methylation analysis. Decision point: if the final trimmed sequence is too short or the analysis stops with 'no CpG sites were found covered by sequencing results', inspect the Trimming plot and lower th_quality_error (e.g. to 0.01, i.e. Q20 / 99% accuracy). Correct application is verified by checking the trimming plot where the quality-score boundary is drawn in orange, the primary-ratio boundary in blue, and the combined final boundary in green, with ends neither over-trimmed nor left low-quality.

## Related tools

- **ABSP** (R tool implementing the quality-score trimming method (th_quality_error) and its combination with primary peak-ratio trimming to produce the final trimmed sequence) — https://github.com/ABSP-methylation-tool/ABSP
- **sangeranalyseR** (Parses .ab1 chromatogram base calls with per-position Phred quality scores and primary/mixed peak ratios feeding the trimming)
- **R** (Language in which ABSP and the trimming thresholds are implemented and executed) — https://www.r-project.org/

## Examples

```
Open ABSP Rproject.Rproject in RStudio, run app.R ("Run App"), then in the "Individual analysis" tab load the .fasta reference and .ab1 sequencing files and inspect the "Trimming plot" in the "Sequencing trimming" tab; adjust th_quality_error at line 607 of ABSP_individual_analysis.Rmd if trimming is too aggressive.
```

## Evaluation signals

- Trimming plot shows the quality-score boundary (orange) nested consistently with the primary-ratio boundary (blue) and the combined final boundary (green)
- Default threshold check: th_quality_error = 0.001 corresponds to base-calling accuracy of 99.9% / Phred Q30; if lowered to 0.01 it must correspond to 99% / Q20
- Final trimmed sequence must not be excessively short: only a few CpG sites covered in the 'Methylation' tab, or the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results', indicates over-trimming
- Final trimmed sequence equals the overlap of the quality-score-trimmed and peak-ratio-trimmed sequences (neither ends trimmed too short nor low-quality ends retained)
- Subsequent CpG methylation percentages are computed on this trimmed sequence only

## Limitations

- The Phred quality-score trimming method can be too aggressive; if so, th_quality_error should be lowered (e.g. from 0.001 to 0.01) in ABSP_individual_analysis.Rmd line 607
- The quality-score method alone is insufficient — ABSP combines it with the primary peak-ratio method (th_mixed_position, th_mixed_perc) because trimming should neither cut the sequence too short nor keep low-quality ends
- If trimming removes too much sequence, the analysis may abort with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'

## Evidence

- [methods] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [methods] the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001: "the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001"
- [readme] The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green).: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)."
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
