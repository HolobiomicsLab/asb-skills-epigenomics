---
name: quality-threshold-filtering-of-sequencing-reads
description: Use when processing raw .ab1 BSP trace files where reads must be trimmed
  of low-quality ends and reads failing bisulfite conversion (non-CpG C-to-T conversion
  rate below 0.9) must be flagged before CpG methylation computation.
when_to_use_negative:
- Input is already trimmed, methylation-called data (e.g. a methylation percentage
  table) — filtering on .ab1 peak heights is unnecessary
- Data are bulk sequencing reads in FASTQ/BAM rather than Sanger-style .ab1 chromatogram
  traces from BSP experiments
- Only visualization or group-comparison of already-computed methylation percentages
  is needed — use the Grouped analysis tab instead of re-running per-read QC
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: ABSP
  role: R tool performing the trimming (Phred quality scores and primary peak ratios)
    and bisulfite conversion-rate QC on .ab1 traces; parameters th_quality_error,
    th_mixed_position, th_mixed_perc are set in ABSP_individual_analysis.Rmd
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP parses trace files, computes conversion
    ratios, and applies the 0.9 acceptance threshold
- name: RStudio
  role: Used to open ABSP Rproject.Rproject and app.R and run the Shiny app whose
    'Sequencing trimming' tab exposes the QC diagnostics
- name: Shiny
  role: Interface hosting the 'Individual analysis' and 'Sequencing trimming' tabs
    used to inspect trimming plots and launch QC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:6b05b87fe1cec9cb43ccc2a2489c45857bd184e45022ac5c2c653e6bdf84263c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - Shiny
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Quality-threshold filtering of sequencing reads

## Summary

Applies ABSP's automated quality control to bisulfite sequencing PCR (.ab1) chromatogram reads: trimming reads by Phred quality scores and primary peak ratios, then accepting only reads whose bisulfite conversion rate over non-CpG cytosines exceeds a 0.9 threshold.

## When to use

Use when processing raw .ab1 BSP trace files where reads must be trimmed of low-quality ends and reads failing bisulfite conversion (non-CpG C-to-T conversion rate below 0.9) must be flagged before CpG methylation computation.

## When NOT to use

- Input is already trimmed, methylation-called data (e.g. a methylation percentage table) — filtering on .ab1 peak heights is unnecessary
- Data are bulk sequencing reads in FASTQ/BAM rather than Sanger-style .ab1 chromatogram traces from BSP experiments
- Only visualization or group-comparison of already-computed methylation percentages is needed — use the Grouped analysis tab instead of re-running per-read QC

## Inputs

- Raw sequencing trace files (.ab1) from direct-BSP or cloning-BSP
- Reference sequence in .fasta format
- Optional reference genome assembly (e.g. GCF_900626175.2_cs10_genomic.fna.gz) for plotting context

## Outputs

- Trimmed per-read sequences with QC flags (pass/fail at the 0.9 conversion threshold)
- Per-read bisulfite conversion rates over non-CpG cytosines (values in [0,1])
- Trimming plot (quality-score and primary-ratio trim boundaries per read)
- Per-read results written to output tables (e.g. conversion_rate_qc.csv) in the 'results' folder

## How to apply

Load .ab1 trace files with the matching reference .fasta into R via ABSP's trace-parsing routines, extracting primary peak heights per position. Trim each read by combining two methods: Phred quality-score trimming (drop positions with base-calling error probability above th_quality_error, default 0.001, i.e. Phred 30) and primary-peak-ratio trimming (a position is non-mixed if its primary peak ratio exceeds th_mixed_position, default 0.75, with at least th_mixed_perc, default 75%, of trimmed positions non-mixed). Next, at each non-CpG cytosine (forward reads) or non-CpG guanine (reverse reads) compute the conversion ratio peakT/(peakC+peakT) or peakA/(peakG+peakA), average over positions, and accept the read only if the rate is at or above the default 0.9 threshold. Rationale: trimming removes low-quality ends while mixed-position and conversion-rate filters prevent erroneous methylation calls from poorly converted or contaminated chromatograms. Inspect the 'Trimming plot' in the 'Sequencing trimming' tab to judge whether either trimming method is too aggressive or too strict before finalizing.

## Related tools

- **ABSP** (R tool performing the trimming (Phred quality scores and primary peak ratios) and bisulfite conversion-rate QC on .ab1 traces; parameters th_quality_error, th_mixed_position, th_mixed_perc are set in ABSP_individual_analysis.Rmd) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP parses trace files, computes conversion ratios, and applies the 0.9 acceptance threshold)
- **RStudio** (Used to open ABSP Rproject.Rproject and app.R and run the Shiny app whose 'Sequencing trimming' tab exposes the QC diagnostics)
- **Shiny** (Interface hosting the 'Individual analysis' and 'Sequencing trimming' tabs used to inspect trimming plots and launch QC)

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # then use the 'Individual analysis' tab with .ab1 + .fasta inputs; adjust th_quality_error (line 607), th_mixed_position (line 612), th_mixed_perc (line 614) in ABSP_individual_analysis.Rmd
```

## Evaluation signals

- Every computed bisulfite conversion rate is numeric and lies within [0, 1]
- Reads with average non-CpG conversion rate >= 0.9 are labeled 'pass'; those below are marked as failing QC
- The output CSV contains exactly one row per processed example read, each with its strand and QC flag
- Trimmed sequence length is sensible: not reduced to near-zero CpG coverage, nor retaining low-quality ends (check the Trimming plot: orange quality-based, blue primary-ratio, green combined trim)
- Analysis aborts with the documented error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.' if over-trimming removed all CpG coverage — this indicates a threshold misconfiguration

## Limitations

- If the Phred-quality trimming is too aggressive, th_quality_error (default 0.001, Phred 30) in ABSP_individual_analysis.Rmd (line 607) must be lowered manually, e.g. to 0.01 (Phred 20)
- If primary-ratio trimming is too strict, th_mixed_position (default 0.75, line 612) and th_mixed_perc (default 75%, line 614) must be adjusted in ABSP_individual_analysis.Rmd
- Over-trimming can yield very short sequences with few covered CpG sites, or abort the analysis when no CpG sites remain covered

## Evidence

- [abstract] It is fully automated and includes a user-friendly interface as a built-in R shiny app, quality control steps and generates publication-ready graphics.: "It is fully automated and includes a user-friendly interface as a built-in R shiny app, quality control steps and generates publication-ready graphics."
- [methods] this is the minimum primary peak ratio for a position to be considered non-mixed: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed"
- [readme] the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001: "consider lowering the minimum base-calling error probability threshold `th_quality_error` in the `ABSP_individual_analysis.Rmd` file (line 607). By default, this is set to 0.001"
- [readme] First, check the “Trimming plot” in the “Sequencing trimming” tab, as this will provide insight into the trimming step results.: "First, check the “Trimming plot” in the “Sequencing trimming” tab, as this will provide insight into the trimming step results."
- [readme] Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.: "the analysis may be aborted with the error message: “Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.”"
