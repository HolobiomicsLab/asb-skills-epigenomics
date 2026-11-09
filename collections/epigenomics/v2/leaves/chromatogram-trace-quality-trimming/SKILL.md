---
name: chromatogram-trace-quality-trimming
description: Use when raw .ab1 Sanger sequencing trace files must be trimmed of low-quality
  and mixed-base ends prior to computing CpG methylation percentages from direct-BSP
  or cloning-BSP data, or when a final trimmed sequence is too short and trimming
  thresholds need adjustment.
when_to_use_negative:
- Input is already a trimmed sequence or a FASTQ/FASTA file rather than a raw .ab1
  chromatogram trace — ABSP trimming operates on trace files with peak-ratio information
- Data are high-throughput short reads (e.g. bisulfite-sequencing FASTQ) rather than
  region-specific BSP chromatogram traces
- The goal is whole-genome methylation profiling; ABSP is designed for region-specific
  CpG methylation from BSP experiments
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: ABSP
  role: R tool implementing the combined quality-score and primary peak-ratio trimming
    of .ab1 traces before CpG methylation computation; thresholds editable in ABSP_individual_analysis.Rmd
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: sangeranalyseR
  role: Parses .ab1 chromatogram base calls with per-position Phred quality scores
    and primary/mixed peak ratios used by the trimming methods
- name: R / RStudio / shiny
  role: Runtime environment; trimming results are inspected in the 'Sequencing trimming'
    tab of the ABSP Shiny app
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:52c5bd1b781b7f17d9511c443e0fbd392789f29335f6eab166f7a260dcce1e93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - Gviz
  - R
  - R / RStudio / shiny
  - RStudio
  - sangeranalyseR
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chromatogram-trace-quality-trimming

## Summary

Trims raw Sanger .ab1 chromatogram reads by combining Phred quality-score trimming (th_quality_error = 0.001, i.e. 99.9% base-calling accuracy / Phred 30) with primary peak-ratio mixed-base trimming (th_mixed_position = 0.75, th_mixed_perc = 75%), as implemented in the ABSP R tool, to yield a reliable trimmed sequence before CpG methylation computation.

## When to use

Use when raw .ab1 Sanger sequencing trace files must be trimmed of low-quality and mixed-base ends prior to computing CpG methylation percentages from direct-BSP or cloning-BSP data, or when a final trimmed sequence is too short and trimming thresholds need adjustment.

## When NOT to use

- Input is already a trimmed sequence or a FASTQ/FASTA file rather than a raw .ab1 chromatogram trace — ABSP trimming operates on trace files with peak-ratio information
- Data are high-throughput short reads (e.g. bisulfite-sequencing FASTQ) rather than region-specific BSP chromatogram traces
- The goal is whole-genome methylation profiling; ABSP is designed for region-specific CpG methylation from BSP experiments

## Inputs

- Raw Sanger sequencing trace files in .ab1 format (e.g. from the ABSP repository 'examples' folder)
- Reference sequence of the studied region in .fasta format (e.g. CDH1 promoter)

## Outputs

- Final trimmed sequence per read (written to trimmed_sequences.fasta)
- Trimming plot showing quality-score (orange), primary-ratio (blue), and combined final (green) trimming boundaries for QC inspection
- Trimmed sequences feeding the downstream per-CpG methylation percentage computation

## How to apply

Parse each .ab1 chromatogram with sangeranalyseR in R to obtain per-position Phred quality scores and primary/mixed peak ratios. Apply quality-score trimming using the minimum base-calling error probability threshold th_quality_error = 0.001 (base-calling accuracy 99.9%, Phred 30, set at line 607 of ABSP_individual_analysis.Rmd), and primary peak-ratio trimming using th_mixed_position = 0.75 (minimum primary peak ratio for a position to be non-mixed, line 612) and th_mixed_perc = 75% (minimum percentage of non-mixed positions, line 614), scanning windows of n = 3 to 15 consecutive non-mixed positions. The final trimmed sequence is obtained by overlapping the sequences trimmed by both methods, which prevents ends from being trimmed too short or left low-quality. Inspect the 'Trimming plot' where orange marks the quality-score boundaries, blue the primary-ratio boundaries, and green the combined final trim; if the final sequence is too short, lower th_quality_error (e.g. to 0.01, corresponding to Phred 20) or relax th_mixed_position. Write the final trimmed sequences to trimmed_sequences.fasta and use them for subsequent CpG methylation level computation.

## Related tools

- **ABSP** (R tool implementing the combined quality-score and primary peak-ratio trimming of .ab1 traces before CpG methylation computation; thresholds editable in ABSP_individual_analysis.Rmd) — https://github.com/ABSP-methylation-tool/ABSP
- **sangeranalyseR** (Parses .ab1 chromatogram base calls with per-position Phred quality scores and primary/mixed peak ratios used by the trimming methods)
- **R / RStudio / shiny** (Runtime environment; trimming results are inspected in the 'Sequencing trimming' tab of the ABSP Shiny app) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
th_quality_error <- 0.001; th_mixed_position <- 0.75; th_mixed_perc <- 75  # ABSP_individual_analysis.Rmd lines 607/612/614, run via the ABSP Shiny app (Run App on app.R)
```

## Evaluation signals

- Trimming plot shows three boundaries (orange quality-score, blue primary-ratio, green combined) and the green final trim is neither too short nor retaining low-quality ends
- Final trimmed sequence equals the overlap of the quality-score-trimmed and peak-ratio-trimmed sequences
- Trimmed read covers CpG sites — analysis must not abort with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Default thresholds respected unless adjusted: th_quality_error = 0.001 (Phred 30), th_mixed_position = 0.75, th_mixed_perc = 75%
- On the EpigenDx CDH1 example, downstream QC outcome is 5/6 forward reads failing trimming/QC, and group statistics reproduce CpG #4 methylation 0.47% ± 0.81% vs 81.44% ± 3.96% (P = 0.00051)

## Limitations

- If the Phred quality-score trimming is too aggressive the sequence may become very short with few CpG sites covered; lower th_quality_error (e.g. to 0.01 for Phred 20)
- If the primary peak-ratio method is too strict, th_mixed_position and th_mixed_perc must be relaxed, otherwise the analysis can stop with no CpG sites covered
- Trimming thresholds are edited manually in ABSP_individual_analysis.Rmd (lines 607/612/614), not exposed in the Shiny UI

## Evidence

- [results] ABSP obtains the final trimmed sequence by overlapping the sequences trimmed by quality score trimming and by mixed base peak trimming: [evidence span withheld — anchor status: not_found]
- [readme] th_quality_error default 0.001: "By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30."
- [readme] th_mixed_position default: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed"
- [methods] trimming plot tabs: "First, check the “Trimming plot” in the “Sequencing trimming” tab"
- [abstract] ABSP input format: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
