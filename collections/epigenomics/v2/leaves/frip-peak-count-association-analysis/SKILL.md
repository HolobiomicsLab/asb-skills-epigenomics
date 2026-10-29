---
name: frip-peak-count-association-analysis
description: Use when you have per-sample ChIP-seq/DNase-seq QC metrics (FRiP, peak
  number, fragment size, PBC) and need to test whether metric associations hold within
  literature-based assay categories rather than globally.
when_to_use_negative:
- Input is raw FASTQ/BAM without run ChiLin QC outputs (no id_frip./id_frag. JSON
  statistics) — run the ChiLin pipeline first to produce FRiP and fragment-size metrics
- The goal is QC of a single sample rather than cross-sample metric association analysis
  across an atlas of many samples
- Samples cannot be assigned to one of the literature-based assay categories, since
  the correlations are category-specific and pooling them would mask the histone-modification
  discrepancy
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: ChiLin
  role: Produces the per-sample QC metrics (FRiP scores via MACS2 peak calling, fragment
    size estimation, PBC) whose JSON outputs are compiled into the flat metrics table
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller used by ChiLin to generate the peak numbers and FRiP scores
    that are correlated
  repo: https://github.com/taoliu/MACS
- name: R
  role: Computes the 13x13 pairwise correlation matrices per assay category and renders
    the 4-panel correlation heatmap PDF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:8205ed4e59d5d1b1bb1c276682de4e9b1e15f38126f36de74be47dc3aeed96c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# frip-peak-count-association-analysis

## Summary

Reproduces the ChiLin quality-atlas finding that FRiP scores correlate with peak number for chromatin accessibility, transcription factor and chromatin regulator samples but not histone modification samples (correlation ~0.37), and that DNase-seq FRiP anti-correlates with estimated fragment size.

## When to use

Use when you have per-sample ChIP-seq/DNase-seq QC metrics (FRiP, peak number, fragment size, PBC) and need to test whether metric associations hold within literature-based assay categories rather than globally.

## When NOT to use

- Input is raw FASTQ/BAM without run ChiLin QC outputs (no id_frip./id_frag. JSON statistics) — run the ChiLin pipeline first to produce FRiP and fragment-size metrics
- The goal is QC of a single sample rather than cross-sample metric association analysis across an atlas of many samples
- Samples cannot be assigned to one of the literature-based assay categories, since the correlations are category-specific and pooling them would mask the histone-modification discrepancy

## Inputs

- Per-sample ChiLin QC JSON files (id_frip., id_pbc., id_frag., id_map., id_rep.) containing FRiP scores, peak numbers, PBC, estimated fragment sizes and mapping rates
- Assay-category labels (literature-based classification: accessibility, TF, chromatin regulator, histone modification, DNase) per sample

## Outputs

- qc_metric_correlations.csv — pairwise correlations of 13 QC metrics within each assay category
- 4-panel correlation heatmap PDF (Figure S3a–d analogue) rendered with the R plotting backend
- Summary values: FRiP–peak-number correlation per category (high for accessibility/TF/chromatin regulator; ~0.37 for histone modification) and DNase FRiP–fragment-size correlation (negative)

## How to apply

First compile the per-sample ChiLin QC JSON statistics (id_frip., id_pbc., id_frag., id_map., id_rep.) into one flat metrics table using the ChiLin compile_database mechanism (github.com/cfce/chilin/blob/master/demo/compile_database.sh). Split the compiled samples into literature-based assay categories: chromatin accessibility, transcription factor, chromatin regulator, histone modification and DNase. Within each category, compute the 13x13 pairwise correlation matrix across QC metrics with R, mirroring the pipeline's historical-comparison R scripts (e.g. id_bwa_compare.R). Extract the FRiP vs peak-number correlation per category: it should be high for accessibility, TF and chromatin-regulator samples, approximately 0.37 for histone modification samples, and DNase FRiP vs estimated fragment size should be negative. Output the correlations as a CSV (qc_metric_correlations.csv) and render a 4-panel correlation heatmap as PDF with the R plotting backend.

## Related tools

- **ChiLin** (Produces the per-sample QC metrics (FRiP scores via MACS2 peak calling, fragment size estimation, PBC) whose JSON outputs are compiled into the flat metrics table) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller used by ChiLin to generate the peak numbers and FRiP scores that are correlated) — https://github.com/taoliu/MACS
- **R** (Computes the 13x13 pairwise correlation matrices per assay category and renders the 4-panel correlation heatmap PDF)

## Examples

```
bash demo/compile_database.sh && Rscript id_bwa_compare.R  # compile ChiLin QC JSONs into a flat table, then compute per-category 13x13 QC metric correlations in R
```

## Evaluation signals

- FRiP vs peak-number correlation is high for chromatin accessibility, transcription factor and chromatin regulator categories
- FRiP vs peak-number correlation for histone modification samples is approximately 0.37
- FRiP vs estimated fragment size correlation for DNase samples is negative (anti-correlation)
- Each category's correlation matrix is 13x13 across QC metrics, matching the compile_database schema of id_frip./id_pbc./id_frag./id_map./id_rep. fields
- qc_metric_correlations.csv and the 4-panel heatmap are produced and mutually consistent, reproducing Additional file 3: Figure S3a–d

## Limitations

- The FRiP–peak-number relationship is assay-type dependent: it does NOT hold for histone modification samples (correlation 0.37), so global pooling across categories is misleading
- Peak numbers and FRiP scores depend on the peak caller settings — ChiLin uses MACS2 (version 2.1.0 in the documented install) as the default, so correlations are not comparable across different peak-calling configurations
- No concrete environment specification is provided for the historical comparison; the discussion notes a reproducibility claim without commands or artifacts for that part

## Evidence

- [results] FRiP scores of chromatin accessibility, transcription factor and chromatin regulator samples are highly correlated with peak number: [evidence span withheld — anchor status: not_found]
- [readme] compile database script: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [methods] FRiP score calculation step: "step 7: calculate FRiP scores for each sample."
- [methods] fragment size estimation step: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [abstract] quality atlas scale: "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
