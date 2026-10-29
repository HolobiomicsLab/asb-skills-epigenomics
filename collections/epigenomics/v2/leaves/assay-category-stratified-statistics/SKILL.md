---
name: assay-category-stratified-statistics
description: Use when you have per-sample ChIP-seq/DNase-seq QC metrics (FRiP, PBC,
  fragment size, mapping rate) and need to test whether metric correlations differ
  across assay categories before pooling them.
when_to_use_negative:
- The analysis goal is overall sample QC rather than inter-metric correlation structure
  — use the per-sample ChiLin QC report instead.
- Samples cannot be reliably assigned to a literature-based assay category, since
  stratification is the core of the method and pooled correlations will be misleading.
- Fewer samples per category than needed to support a stable 13x13 pairwise correlation
  matrix.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: ChiLin
  role: Produces the per-sample QC JSON statistics (id_frip., id_pbc., id_frag., id_map.,
    id_rep.) and provides the compile_database mechanism used to build the flat metrics
    table
  repo: https://github.com/cfce/chilin
- name: R
  role: Computes the 13x13 pairwise QC-metric correlation matrices per assay category
    and renders the 4-panel correlation heatmap PDF, following the pipeline's historical-comparison
    R scripts (id_bwa_compare.R)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:f4b48c3588834bc83cd4624de382a47e725e06960d3fc9a9152f67787e084bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# assay-category-stratified-statistics

## Summary

Compile per-sample QC metrics from ChiLin JSON outputs into a flat table, stratify samples into literature-based assay categories (accessibility, TF, chromatin regulator, histone modification, DNase), and compute within-category pairwise correlations (e.g., FRiP vs peak number, DNase FRiP vs fragment size) to expose relationships that differ by assay type.

## When to use

Use when you have per-sample ChIP-seq/DNase-seq QC metrics (FRiP, PBC, fragment size, mapping rate) and need to test whether metric correlations differ across assay categories before pooling them.

## When NOT to use

- The analysis goal is overall sample QC rather than inter-metric correlation structure — use the per-sample ChiLin QC report instead.
- Samples cannot be reliably assigned to a literature-based assay category, since stratification is the core of the method and pooled correlations will be misleading.
- Fewer samples per category than needed to support a stable 13x13 pairwise correlation matrix.

## Inputs

- Per-sample ChiLin QC JSON statistic files (id_frip., id_pbc., id_frag., id_map., id_rep.)
- Sample-to-assay-category annotation (accessibility, TF, chromatin regulator, histone modification, DNase)
- Compiled flat QC metrics table with 13 QC metrics per sample (e.g., FRiP, peak number, PBC, estimated fragment size, mapping rate)

## Outputs

- qc_metric_correlations.csv containing per-category pairwise QC metric correlations
- 4-panel correlation heatmap PDF (Figure S3a-d analogue) rendered with the R plotting backend

## How to apply

First compile the per-sample ChiLin QC JSON statistics (id_frip., id_pbc., id_frag., id_map., id_rep.) into one flat metrics table using the ChiLin QC summary/compile_database mechanism. Next, split samples into the literature-based assay categories: chromatin accessibility, transcription factor, chromatin regulator, histone modification, and DNase. Within each category, compute the 13x13 pairwise correlation matrix across QC metrics with R, in the same style as the pipeline's historical-comparison R scripts (id_bwa_compare.R). Extract the FRiP vs peak-number correlation for accessibility, TF, and chromatin-regulator samples (expected high), confirm the histone-modification correlation is ~0.37, and for DNase samples compute FRiP vs estimated fragment size, which should be anti-correlated. The rationale is that pooling categories masks assay-specific behavior, as demonstrated by the histone-modification FRiP–peak-number correlation of only 0.37 versus strong correlations in other categories.

## Related tools

- **ChiLin** (Produces the per-sample QC JSON statistics (id_frip., id_pbc., id_frag., id_map., id_rep.) and provides the compile_database mechanism used to build the flat metrics table) — https://github.com/cfce/chilin
- **R** (Computes the 13x13 pairwise QC-metric correlation matrices per assay category and renders the 4-panel correlation heatmap PDF, following the pipeline's historical-comparison R scripts (id_bwa_compare.R))

## Examples

```
Rscript id_bwa_compare.R qc_metric_correlations.csv
```

## Evaluation signals

- FRiP vs peak-number correlation is high for chromatin accessibility, transcription factor, and chromatin regulator categories
- Histone-modification FRiP vs peak-number correlation is approximately 0.37
- DNase FRiP vs estimated fragment size correlation is negative (anti-correlated)
- qc_metric_correlations.csv is produced with a 13x13 pairwise correlation matrix per assay category
- The rendered heatmap matches the structure of Additional file 3: Figure S3a-d

## Limitations

- Correlations are computed on sub-sampled BAM-derived metrics; ChiLin sub-samples BAM files for fragment size estimation and PBC evaluation, which introduces sampling variability.
- The histone-modification category behaves differently from other assay categories (correlation 0.37), so conclusions from one category cannot be transferred to another.
- The article notes the reproducibility claim without concrete commands or environment specification; exact reproduction may require the specific ChiLin version and R scripts used historically.

## Evidence

- [results] FRiP scores of chromatin accessibility, transcription factor and chromatin regulator samples are highly correlated with peak number, but this is not true for histone modification samples (correlation 0.37).: [evidence span withheld — anchor status: not_found]
- [abstract] ChiLin generates comprehensive quality control reports that include comparisons with historical data: "ChiLin generates comprehensive quality control reports that include comparisons with historical data"
- [readme] post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [methods] step 7: calculate FRiP scores for each sample.: "step 7: calculate FRiP scores for each sample."
- [methods] sub-sample bam files and do macs2 fragment size estimation.: "step 3: sub-sample bam files and do macs2 fragment size estimation."
