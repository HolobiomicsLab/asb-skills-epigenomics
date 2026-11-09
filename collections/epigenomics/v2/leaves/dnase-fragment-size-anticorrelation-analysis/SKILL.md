---
name: dnase-fragment-size-anticorrelation-analysis
description: Use when a compiled ChIP-seq/DNase-seq QC metrics table (FRiP, PBC, fragment
  size, mapping rate, peak number) needs category-wise correlation analysis to validate
  quality-atlas expectations or benchmark a new pipeline against the ChiLin reference
  results.
when_to_use_negative:
- Input data are raw FASTQ/BAM files rather than compiled QC metric tables — run the
  ChiLin pipeline (mapping, macs2 fragment size estimation, PBC, FRiP calculation)
  first.
- The goal is single-sample QC judgment rather than population-level metric correlation
  structure across assay categories.
- Samples lack reliable assay-category labels, since the key result (histone-modification
  correlation ~0.37 vs high correlations elsewhere) depends on correct stratification.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3574
tools:
- name: ChiLin
  role: Produces the per-sample QC JSON statistics (FRiP, PBC, fragment size, mapping
    rate) and the compile_database mechanism that flattens them into the metrics table
    used for correlation analysis
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller whose sub-sampled BAM fragment size estimation supplies
    the estimated fragment-size metric used in the DNase anti-correlation test
  repo: https://github.com/taoliu/MACS
- name: BWA
  role: Default mapper in the ChiLin pipeline (step 2) generating the BAM files and
    mapping-rate metrics underlying the QC table
  repo: https://github.com/lh3/bwa
- name: R
  role: Computes the 13x13 pairwise correlation matrices per assay category and renders
    the 4-panel heatmap PDF, in the style of id_bwa_compare.R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:f37959c0713ab99eb8c29cfceec1e2658269571401f226a4490ae110501cbe16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - BWA
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# dnase-fragment-size-anticorrelation-analysis

## Summary

Reproduce the QC-metric correlation structure from the ChiLin quality atlas: compute per-category pairwise correlations between FRiP scores, peak numbers, PBC, mapping rate and estimated fragment size, confirming that FRiP correlates with peak number for accessibility/TF/chromatin-regulator samples but not for histone modification samples (~0.37), and that DNase-seq FRiP anti-correlates with estimated fragment size.

## When to use

Use when a compiled ChIP-seq/DNase-seq QC metrics table (FRiP, PBC, fragment size, mapping rate, peak number) needs category-wise correlation analysis to validate quality-atlas expectations or benchmark a new pipeline against the ChiLin reference results.

## When NOT to use

- Input data are raw FASTQ/BAM files rather than compiled QC metric tables — run the ChiLin pipeline (mapping, macs2 fragment size estimation, PBC, FRiP calculation) first.
- The goal is single-sample QC judgment rather than population-level metric correlation structure across assay categories.
- Samples lack reliable assay-category labels, since the key result (histone-modification correlation ~0.37 vs high correlations elsewhere) depends on correct stratification.

## Inputs

- Per-sample ChiLin QC JSON statistics files (id_frip., id_pbc., id_frag., id_map., id_rep.)
- Flat compiled QC metrics table produced by ChiLin compile_database (id_bwa, id_frip, id_pbc, id_frag, id_map columns)
- Assay-category annotation per sample (literature-based: accessibility, TF, chromatin regulator, histone modification, DNase)

## Outputs

- qc_metric_correlations.csv (per-category pairwise QC-metric correlation values)
- 4-panel correlation heatmap PDF (Figure S3a–d analogue)

## How to apply

First compile the per-sample ChiLin QC JSON statistics (id_frip., id_pbc., id_frag., id_map., id_rep.) into one flat metrics table using the ChiLin QC summary/compile_database mechanism (demo/compile_database.sh). Split samples into literature-based assay categories (chromatin accessibility, transcription factor, chromatin regulator, histone modification, DNase). Within each category, compute a 13x13 pairwise correlation matrix across QC metrics with R, in the same style as the pipeline's historical-comparison R scripts (id_bwa_compare.R). Extract the FRiP vs peak-number correlations for accessibility, TF and chromatin-regulator categories and confirm they are high; confirm the histone-modification FRiP–peak-number correlation is approximately 0.37; restrict to DNase samples and confirm FRiP vs estimated (macs2-derived) fragment size is negative. Write qc_metric_correlations.csv and render a 4-panel correlation heatmap (Additional file 3, Figure S3a–d analogue) as PDF with the R plotting backend.

## Related tools

- **ChiLin** (Produces the per-sample QC JSON statistics (FRiP, PBC, fragment size, mapping rate) and the compile_database mechanism that flattens them into the metrics table used for correlation analysis) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller whose sub-sampled BAM fragment size estimation supplies the estimated fragment-size metric used in the DNase anti-correlation test) — https://github.com/taoliu/MACS
- **BWA** (Default mapper in the ChiLin pipeline (step 2) generating the BAM files and mapping-rate metrics underlying the QC table) — https://github.com/lh3/bwa
- **R** (Computes the 13x13 pairwise correlation matrices per assay category and renders the 4-panel heatmap PDF, in the style of id_bwa_compare.R)

## Examples

```
bash demo/compile_database.sh && Rscript id_bwa_compare.R  # compile QC JSON stats, then compute per-category 13x13 correlations and render Figure S3a-d heatmap
```

## Evaluation signals

- FRiP vs peak-number correlation is high for chromatin accessibility, TF, and chromatin regulator categories.
- Histone-modification FRiP vs peak-number correlation is approximately 0.37 (not high), distinguishing it from other categories.
- DNase-seq FRiP vs estimated fragment size correlation is negative (anti-correlation).
- Each category yields a 13x13 pairwise correlation matrix across QC metrics, and qc_metric_correlations.csv is populated for all five categories.
- Rendered 4-panel heatmap PDF matches the structure of Additional file 3: Figure S3a–d.

## Limitations

- The FRiP–peak-number relationship does not hold for histone modification samples (correlation only 0.37), so aggregate across-category correlations would mask this heterogeneity.
- The relationship is anchored to the 23,677-sample quality atlas compiled from eight literature-based categories; results depend on consistent category assignment and the same QC metric definitions (ChiLin id_frip/id_pbc/id_frag/id_map).
- No software version is stated for ChiLin in the discussion section, so exact reproduction may require inspecting the repository commit rather than a pinned release.

## Evidence

- [other] FRiP scores of chromatin accessibility, transcription factor and chromatin regulator samples are highly correlated with peak number, but this is not true for histone modification samples (correlation 0.37).: [evidence span withheld — anchor status: not_found]
- [methods] step 3: sub-sample bam files and do macs2 fragment size estimation.: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [abstract] We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users.: "We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users."
- [intro] post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [methods] step 7: calculate FRiP scores for each sample.: "step 7: calculate FRiP scores for each sample."
