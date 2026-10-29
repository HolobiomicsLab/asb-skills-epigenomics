---
name: qc-metric-pairwise-correlation-analysis
description: Use when a batch of ChIP-seq/DNase-seq samples has per-sample ChiLin
  QC JSON outputs and you need to determine which QC metrics covary within each literature-based
  assay category.
when_to_use_negative:
- Input is raw sequencing data with no per-sample ChiLin QC JSON outputs yet — run
  the ChiLin pipeline (mapping, macs2 fragment size estimation, PBC, FRiP) first.
- The question is about absolute QC quality thresholds for a single sample rather
  than covariation structure of QC metrics across a sample collection.
- Samples cannot be assigned to a literature-based assay category, since correlations
  are computed and interpreted per category.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3574
tools:
- name: ChiLin
  role: Generates the per-sample QC metrics (FRiP, PBC, fragment size, mapping, replicate
    statistics) and provides the compile_database mechanism to flatten QC JSONs into
    a metrics table.
  repo: https://github.com/cfce/chilin
- name: R
  role: Computes the 13x13 pairwise correlation matrices per assay category and renders
    the 4-panel correlation heatmap PDF, following the pipeline's historical-comparison
    R scripts (id_bwa_compare.R).
- name: MACS2
  role: Default peak caller whose fragment size estimation and peak numbers feed the
    FRiP and fragment-size correlation analysis.
  repo: https://github.com/taoliu/MACS/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:cc4e0447dc95a8e54af0bc62cfc36c2e6a5449de4b643976b639171434d843f3
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
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# qc-metric-pairwise-correlation-analysis

## Summary

Compile per-sample ChiLin QC statistics (FRiP, PBC, fragment size, mapping, replicate metrics) into a flat metrics table, split samples into assay categories, and compute 13x13 pairwise correlation matrices with R to reproduce the paper's QC-metric correlation structure (e.g., FRiP vs peak number).

## When to use

Use when a batch of ChIP-seq/DNase-seq samples has per-sample ChiLin QC JSON outputs and you need to determine which QC metrics covary within each literature-based assay category.

## When NOT to use

- Input is raw sequencing data with no per-sample ChiLin QC JSON outputs yet — run the ChiLin pipeline (mapping, macs2 fragment size estimation, PBC, FRiP) first.
- The question is about absolute QC quality thresholds for a single sample rather than covariation structure of QC metrics across a sample collection.
- Samples cannot be assigned to a literature-based assay category, since correlations are computed and interpreted per category.

## Inputs

- Per-sample ChiLin QC JSON statistics (id_frip., id_pbc., id_frag., id_map., id_rep.)
- Assay-category labels for each sample (accessibility, TF, chromatin regulator, histone modification, DNase)
- Compiled QC database from ChiLin compile_database mechanism

## Outputs

- qc_metric_correlations.csv (per-category FRiP vs peak-number and DNase FRiP vs fragment-size correlations)
- 13x13 pairwise correlation matrices across QC metrics per assay category
- 4-panel correlation heatmap PDF (Figure S3a-d analogue)

## How to apply

First run ChiLin on all samples so that per-sample QC JSON files (id_frip., id_pbc., id_frag., id_map., id_rep.) are generated, then flatten them into one table using the compile_database.sh mechanism from github.com/cfce/chilin. Split the compiled samples into the literature-based categories used by the paper: chromatin accessibility, transcription factor, chromatin regulator, histone modification, and DNase. Within each category, compute a 13x13 pairwise correlation matrix across the QC metrics in R, in the same style as the pipeline's historical-comparison R scripts (e.g., id_bwa_compare.R). Extract the FRiP vs peak-number correlation per category: accessibility, TF and chromatin-regulator samples should show high correlation, while histone modification samples should be weak (~0.37). Restrict to DNase samples and check the FRiP vs estimated-fragment-size (macs2 fragment size estimation) anti-correlation. Write the result as qc_metric_correlations.csv and render a 4-panel correlation heatmap (Figure S3a-d analogue) as PDF with the R plotting backend.

## Related tools

- **ChiLin** (Generates the per-sample QC metrics (FRiP, PBC, fragment size, mapping, replicate statistics) and provides the compile_database mechanism to flatten QC JSONs into a metrics table.) — https://github.com/cfce/chilin
- **R** (Computes the 13x13 pairwise correlation matrices per assay category and renders the 4-panel correlation heatmap PDF, following the pipeline's historical-comparison R scripts (id_bwa_compare.R).)
- **MACS2** (Default peak caller whose fragment size estimation and peak numbers feed the FRiP and fragment-size correlation analysis.) — https://github.com/taoliu/MACS/

## Examples

```
git clone https://github.com/cfce/chilin; bash demo/compile_database.sh && Rscript id_bwa_compare.R
```

## Evaluation signals

- FRiP vs peak-number correlation is high for chromatin accessibility, TF and chromatin-regulator categories.
- Histone-modification FRiP vs peak-number correlation is approximately 0.37.
- DNase-restricted FRiP vs estimated-fragment-size correlation is negative (anti-correlation reproduced).
- Compiled metrics table has one row per sample with all 13 QC metrics populated from id_frip./id_pbc./id_frag./id_map./id_rep. JSONs.
- Each per-category correlation matrix is exactly 13x13 and heatmap renders as a 4-panel PDF matching the Figure S3a-d analogue.

## Limitations

- The FRiP–peak-number relationship differs by assay category: it does not hold for histone modification samples (correlation 0.37), so a single pooled correlation across all categories is misleading.
- ChiLin's R-based QC tooling assumes users are familiar with R programming to utilize them.
- No software version is stated for ChiLin, and the reproducibility claim lacks concrete commands, environment specification, or output artifacts in the discussion section.

## Evidence

- [results] FRiP scores of chromatin accessibility, transcription factor and chromatin regulator samples are highly correlated with peak number, but this is not true for histone modification samples (correlation 0.37).: [evidence span withheld — anchor status: not_found]
- [methods] FRiP score calculation: "step 7: calculate FRiP scores for each sample."
- [intro] compile_database.sh: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [readme] Docker version: "It is a pipeline for processing ChIP-seq and DNase-seq data."
- [abstract] comparisons with historical data: "ChiLin generates comprehensive quality control reports that include comparisons with historical data"
