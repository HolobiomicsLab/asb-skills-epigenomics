---
name: bisulfite-conversion-rate-quality-control
description: Use when analyzing direct-BSP or cloning-BSP .ab1 chromatogram traces
  in ABSP and you must verify bisulfite conversion efficiency at non-CpG positions
  before computing or interpreting CpG methylation percentages.
when_to_use_negative:
- You need CpG methylation percentages, lollipop plots, or between-group comparative
  statistics — use ABSP's CpG methylation computation and Grouped analysis steps,
  not this QC filter.
- Input data are next-generation sequencing reads (FASTQ/BAM) — ABSP operates on raw
  Sanger sequencing trace files (.ab1) from BSP experiments.
- The question is reference-genome availability for genomic plots — the reference
  genome is only needed for visualization and plays no role in conversion-rate or
  methylation computation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: ABSP
  role: R tool whose individual analysis (ABSP_individual_analysis.Rmd) computes per-position
    non-CpG conversion ratios, averages them per read, and applies the 0.9 default
    QC threshold
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment for ABSP; trace-parsing, alignment, and ratio computations
    are executed in R
  repo: https://www.r-project.org/
- name: Shiny
  role: Built-in Shiny app interface (Individual analysis tab) through which the .ab1
    and .fasta inputs are supplied and the QC-computing analysis is launched
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:475d44dd1a0a76e2de81edf5656315081ca036fc289e779789c06a6ff79dff62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# bisulfite-conversion-rate-quality-control

## Summary

ABSP's quality-control step that quantifies bisulfite conversion efficiency from Sanger .ab1 trace files by measuring C-to-T conversion at non-CpG cytosines (A-to-G on reverse reads), accepting reads whose average conversion rate exceeds a threshold (0.9 by default) before CpG methylation percentages are trusted.

## When to use

Use when analyzing direct-BSP or cloning-BSP .ab1 chromatogram traces in ABSP and you must verify bisulfite conversion efficiency at non-CpG positions before computing or interpreting CpG methylation percentages.

## When NOT to use

- You need CpG methylation percentages, lollipop plots, or between-group comparative statistics — use ABSP's CpG methylation computation and Grouped analysis steps, not this QC filter.
- Input data are next-generation sequencing reads (FASTQ/BAM) — ABSP operates on raw Sanger sequencing trace files (.ab1) from BSP experiments.
- The question is reference-genome availability for genomic plots — the reference genome is only needed for visualization and plays no role in conversion-rate or methylation computation.

## Inputs

- Primary sequencing trace files (.ab1) from bisulfite sequencing PCR
- Reference sequence in .fasta format
- Conversion-rate acceptance threshold (default 0.9, user-provided)

## Outputs

- conversion_rate_qc.csv with one row per processed read: per-read bisulfite conversion rate, read strand, and QC flag ('pass' if rate >= 0.9)
- Per-position C-to-T (or A-to-G) conversion ratios at non-CpG sites used in QC

## How to apply

After ABSP's sequencing trimming (Phred-score method with `th_quality_error` default 0.001, plus primary-peak-ratio method with `th_mixed_position` 0.75 and `th_mixed_perc` 75%), align each trimmed read to the reference .fasta and identify non-CpG cytosines. For forward reads compute the per-position C-to-T ratio peakT/(peakC+peakT) from primary peak heights; for reverse reads (or reverse-complement of forward-aligned reads) compute peakA/(peakG+peakA) at non-CpG guanine positions. Average the per-position ratios into one bisulfite conversion rate per read and apply the acceptance threshold, 0.9 by default: reads below 0.9 fail quality control. This is implemented in ABSP's individual analysis (ABSP_individual_analysis.Rmd); the rationale is that unmethylated non-CpG cytosines should be fully converted by bisulfite treatment, so low conversion rates indicate incomplete bisulfite reaction and unreliable methylation calls.

## Related tools

- **ABSP** (R tool whose individual analysis (ABSP_individual_analysis.Rmd) computes per-position non-CpG conversion ratios, averages them per read, and applies the 0.9 default QC threshold) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment for ABSP; trace-parsing, alignment, and ratio computations are executed in R) — https://www.r-project.org/
- **Shiny** (Built-in Shiny app interface (Individual analysis tab) through which the .ab1 and .fasta inputs are supplied and the QC-computing analysis is launched)

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # then Individual analysis tab: load .ab1 traces + reference .fasta; ABSP_individual_analysis.Rmd computes conversion_rate_qc with default 0.9 threshold
```

## Evaluation signals

- Each computed bisulfite conversion rate is a numeric value between 0 and 1.
- Reads with an average rate at or above the default 0.9 threshold are labeled 'pass'; reads below 0.9 are marked as failing QC.
- conversion_rate_qc.csv contains exactly one row per processed example read, with strand and QC flag columns populated.
- Per-position ratios are computed only at non-CpG C positions on forward reads (peakT/(peakC+peakT)) and non-CpG G positions on reverse reads (peakA/(peakG+peakA)), excluding CpG sites.
- Rates are derived from primary peak heights of the .ab1 traces after ABSP's trimming step, not from base-call strings.

## Limitations

- Over-aggressive trimming can leave a very short sequence covering only a few CpG sites, or abort the analysis with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Trimming thresholds (th_quality_error default 0.001, th_mixed_position default 0.75, th_mixed_perc default 75%) may need manual adjustment in ABSP_individual_analysis.Rmd; mixed-position filtering relies on a minimum primary peak ratio of 0.75 for a position to be considered non-mixed.
- The threshold is user-provided (0.9 by default); reads just below the cutoff reflect borderline bisulfite conversion and the choice of cutoff affects QC stringency.

## Evidence

- [results] ABSP computes the bisulfite conversion rate for each cytosine position outside CpG sites, and the average rate on the sequence must be higher than a provided threshold, which is 0.9 by default.: [evidence span withheld — anchor status: not_found]
- [abstract] It is fully automated and includes a user-friendly interface as a built-in R shiny app, quality control steps and generates publication-ready graphics.: "It is fully automated and includes a user-friendly interface as a built-in R shiny app, quality control steps and generates publication-ready graphics."
- [readme] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format.: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
- [methods] th_mixed_position (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed: "this is the minimum primary peak ratio for a position to be considered non-mixed"
