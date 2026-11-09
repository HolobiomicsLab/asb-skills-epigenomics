---
name: primary-peak-ratio-mixed-base-trimming
description: Use when processing .ab1 Sanger chromatogram traces in ABSP whose ends
  contain mixed-base positions, and the goal is to define trimming boundaries before
  computing CpG methylation percentages.
when_to_use_negative:
- Input is not raw .ab1 chromatogram trace data (e.g. FASTQ NGS reads or pre-trimmed
  consensus sequences) — ABSP trimming operates on trace-file base calls and peak
  ratios
- 'The reference genome is missing: ABSP can still compute methylation percentages
  without it; only genomic plot visualization needs the genome, so trimming should
  not be skipped or substituted for that reason'
- The analysis goal is comparing methylation between sample groups (lollipop plots,
  statistics) rather than defining per-read trimmed sequences
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ABSP
  role: Implements the primary peak ratio trimming method and combines it with quality-score
    trimming to produce the final trimmed sequence
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Language in which ABSP's trimming thresholds and trimming logic are scripted
    (ABSP_individual_analysis.Rmd)
- name: Shiny
  role: User interface exposing the 'Sequencing trimming' tab where the trimming plot
    is inspected
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:03765087cc73acc02c7e2a5aaf0f803daf44fbe9246beb6e33d4d83f82b2ee58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  - sangeranalyseR
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# primary-peak-ratio-mixed-base-trimming

## Summary

A chromatogram trimming method from the ABSP R tool that removes low-quality ends of .ab1 bisulfite sequencing PCR reads by scanning for windows of consecutive non-mixed base positions, using primary peak ratio thresholds, and combines this with Phred-quality trimming to yield the final trimmed sequence used for CpG methylation computation.

## When to use

Use when processing .ab1 Sanger chromatogram traces in ABSP whose ends contain mixed-base positions, and the goal is to define trimming boundaries before computing CpG methylation percentages.

## When NOT to use

- Input is not raw .ab1 chromatogram trace data (e.g. FASTQ NGS reads or pre-trimmed consensus sequences) — ABSP trimming operates on trace-file base calls and peak ratios
- The reference genome is missing: ABSP can still compute methylation percentages without it; only genomic plot visualization needs the genome, so trimming should not be skipped or substituted for that reason
- The analysis goal is comparing methylation between sample groups (lollipop plots, statistics) rather than defining per-read trimmed sequences

## Inputs

- Raw Sanger sequencing trace files (.ab1 chromatograms) with base calls and peak information
- Reference sequence in .fasta format (for the surrounding ABSP individual analysis)
- Threshold parameters: th_mixed_position (default 0.75), th_mixed_perc (default 75%), th_quality_error (default 0.001)

## Outputs

- Final trimmed nucleotide sequence (overlap of quality-score and primary-peak-ratio trimming) used for CpG methylation percentage computation
- Trimming plot showing quality-score (orange), primary-ratio (blue), and combined final (green) trimming boundaries
- trimmed_sequences.fasta per read

## How to apply

Within ABSP (R/sangeranalyseR), parse the .ab1 trace to obtain per-position primary/mixed peak ratios, then apply the primary peak ratio trimming method with `th_mixed_position` = 0.75 (minimum primary peak ratio for a position to be considered non-mixed) and `th_mixed_perc` = 75% (minimum percentage of non-mixed positions in the trimmed sequence), scanning windows of 3–15 consecutive non-mixed positions. Rationale: positions with low primary peak ratios indicate mixed bases from sloppy ends or overlapping signals and must be excluded from downstream methylation calls. The result of this method is overlapped with the Phred quality-score trimming result (th_quality_error = 0.001, i.e. 99.9% base-calling accuracy / Phred 30), and the intersection of the two trims produces the final trimmed sequence (shown in green on the trimming plot) used for CpG methylation level computation. Decision point: if this method is too strict and trims the sequence too short, relax `th_mixed_position` (or `th_mixed_perc`) in the `ABSP_individual_analysis.Rmd` file.

## Related tools

- **ABSP** (Implements the primary peak ratio trimming method and combines it with quality-score trimming to produce the final trimmed sequence) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Language in which ABSP's trimming thresholds and trimming logic are scripted (ABSP_individual_analysis.Rmd))
- **Shiny** (User interface exposing the 'Sequencing trimming' tab where the trimming plot is inspected)

## Evaluation signals

- The trimming plot displays all three boundaries: quality-score (orange), primary-ratio (blue), and combined final (green) trimming points per read
- The final trimmed sequence corresponds to the overlap of the quality-score-trimmed and primary-ratio-trimmed sequences, not to either method alone
- The trimmed sequence is neither too short nor retains low-quality ends, per the stated purpose of ABSP trimming
- Downstream, the 'Methylation' tab covers CpG sites; aborting with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.' signals over-aggressive trimming
- Default thresholds respected unless deliberately relaxed: th_mixed_position = 0.75, th_mixed_perc = 75%, th_quality_error = 0.001 (Phred 30)

## Limitations

- If the primary peak ratio trimming is too strict or quality trimming too aggressive, the final sequence may be very short with only a few CpG sites covered, or the analysis may abort with no CpG sites covered; thresholds must then be lowered/adjusted in ABSP_individual_analysis.Rmd (lines 607–614)
- Lowering th_quality_error to 0.01 corresponds to a base-calling accuracy of 99% (Phred 20), trading stricter trimming for longer retained sequence
- On Linux, first-launch package installation may take considerable time, delaying the ABSP workflow

## Evidence

- [readme] The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green).: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)."
- [methods] th_mixed_position is the minimum primary peak ratio for a position to be considered non-mixed: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed"
- [readme] If the trimming method based on Primary peak ratios is too strict, you can adjust the following thresholds: `th_mixed_position`: "If the trimming method based on Primary peak ratios is too strict, you can adjust the following thresholds: `th_mixed_position`"
- [methods] First, check the “Trimming plot” in the “Sequencing trimming” tab: "First, check the “Trimming plot” in the “Sequencing trimming” tab"
