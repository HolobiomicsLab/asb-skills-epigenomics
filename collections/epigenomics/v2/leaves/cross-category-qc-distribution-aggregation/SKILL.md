---
name: cross-category-qc-distribution-aggregation
description: 'Use when benchmarking ChIP-seq/DNase-seq QC: you need cumulative distributions
  and cutoffs for read-layer metrics (FastQC score, uniquely mapped ratio, PBC) across
  many public samples, or to place a new sample against the historical quality atlas.'
when_to_use_negative:
- The question concerns signal-layer QC (replicate peak overlap/correlation, FRiP
  scores, PhastCons conservation, motif analysis — ChiLin steps 5–12) rather than
  read-layer metrics.
- Input is a single small dataset where a per-sample ChiLin QC report suffices; category-level
  aggregation over 23,677 historical samples adds no value.
- The assay falls outside ChIP-seq/DNase-seq — the cutoffs were derived from, and
  are suggested as usable for, ChIP-seq data.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: ChiLin
  role: Runs the QC steps (FastQC evaluation, bwa mapping, PBC evaluation on subsampled
    BAMs) that generate the per-sample id_fastqc., id_map., id_pbc. records aggregated
    into the category distributions
  repo: https://github.com/cfce/chilin
- name: FastQC
  role: 'ChiLin step 1: per-sample sequence quality score evaluation feeding the FastQC
    score distribution'
- name: bwa
  role: ChiLin step 2 default mapper producing the BAM files from which uniquely mapped
    read ratios and read depths are computed
  repo: https://github.com/lh3/bwa
- name: samtools
  role: BAM processing supporting mapping ratio statistics and PBC evaluation during
    ChiLin QC steps
- name: R
  role: Runs id_bwa_compare.R to render the comparison of new samples against the
    historical QC distribution (id_bwa_compare.pdf)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:7b94b6a04c8ace085a2f193a0a2c63ba2edc13d30191accd1b683fd25aed630c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - FastQC
  - R
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cross-category-qc-distribution-aggregation

## Summary

Aggregates per-sample read-layer QC metrics — FastQC sequence quality score, uniquely mapped read ratio, and PBC — from 23,677 public ChIP-seq/DNase-seq samples (11,265 datasets) across eight literature-based classified categories into cumulative distributions. It supports assay-insensitive quality cutoffs (>85% of samples pass quality score >25, uniquely mapped ratio >0.5, PBC >0.8) and median depths (25M ChIP-seq, 40M DNase-seq).

## When to use

Use when benchmarking ChIP-seq/DNase-seq QC: you need cumulative distributions and cutoffs for read-layer metrics (FastQC score, uniquely mapped ratio, PBC) across many public samples, or to place a new sample against the historical quality atlas.

## When NOT to use

- The question concerns signal-layer QC (replicate peak overlap/correlation, FRiP scores, PhastCons conservation, motif analysis — ChiLin steps 5–12) rather than read-layer metrics.
- Input is a single small dataset where a per-sample ChiLin QC report suffices; category-level aggregation over 23,677 historical samples adds no value.
- The assay falls outside ChIP-seq/DNase-seq — the cutoffs were derived from, and are suggested as usable for, ChIP-seq data.

## Inputs

- Per-sample FASTQ files for the 23,677 public ChIP-seq/DNase-seq samples (11,265 datasets)
- BAM files produced by ChiLin step 2 bwa mapping (input to PBC evaluation on subsampled BAMs and depth statistics)
- Per-sample ChiLin QC JSON records (id_fastqc., id_map., id_pbc.)
- Category labels mapping each sample to one of the eight literature-based classified categories

## Outputs

- qc_metric_summary.csv — aggregated category-level cutoff and median-depth table
- Cumulative distribution statistics per category (fraction passing quality score >25, uniquely mapped ratio >0.5, PBC >0.8)
- id_bwa_compare.pdf — plot comparing a new sample against the historical QC distribution

## How to apply

Run the ChiLin QC steps per sample: step 1 FastQC sequence quality evaluation, step 2 bwa (default) mapping — which cannot be skipped because it produces the necessary BAM files — and step 4 PBC evaluation on subsampled BAM files, yielding per-sample id_fastqc., id_map., and id_pbc. records. Parse every per-sample QC JSON record (fastqc evaluation, mapping ratio statistics, PBC score) with Python and assign each of the 23,677 samples (11,265 datasets) to one of the eight literature-based classified categories. Per category, compute the distributions of FastQC quality scores, uniquely mapped ratios, and PBC values, plus the cumulative fraction of samples passing each cutoff (quality score >25, unique mapped ratio >0.5, PBC >0.8) and the median read depth (25M ChIP-seq, 40M DNase-seq). Write the aggregated category-level cutoff and median-depth table to qc_metric_summary.csv. Finally, render a new sample against the historical QC distribution with the id_bwa_compare R script (id_bwa_compare.R -> id_bwa_compare.pdf) and flag any category that deviates from the reported cumulative cutoffs.

## Related tools

- **ChiLin** (Runs the QC steps (FastQC evaluation, bwa mapping, PBC evaluation on subsampled BAMs) that generate the per-sample id_fastqc., id_map., id_pbc. records aggregated into the category distributions) — https://github.com/cfce/chilin
- **FastQC** (ChiLin step 1: per-sample sequence quality score evaluation feeding the FastQC score distribution)
- **bwa** (ChiLin step 2 default mapper producing the BAM files from which uniquely mapped read ratios and read depths are computed) — https://github.com/lh3/bwa
- **samtools** (BAM processing supporting mapping ratio statistics and PBC evaluation during ChiLin QC steps)
- **R** (Runs id_bwa_compare.R to render the comparison of new samples against the historical QC distribution (id_bwa_compare.pdf))

## Examples

```
Rscript id_bwa_compare.R
```

## Evaluation signals

- More than 85% of aggregated samples show FastQC sequence quality score >25
- More than 85% of samples show uniquely mapped read ratio >0.5 and PBC >0.8
- Median read depth lands at ~25M reads for ChIP-seq and ~40M for DNase-seq categories
- qc_metric_summary.csv contains one aggregated row per literature-based category with cutoff and median-depth values
- id_bwa_compare.R executes and writes id_bwa_compare.pdf placing the new sample within the historical QC distribution

## Limitations

- Cutoffs are suggested as usable for ChIP-seq data; the atlas covers only the eight literature-based classified categories of ChIP-seq and DNase-seq samples
- PBC evaluation operates on subsampled BAM files, so absolute metric values depend on the subsampling scheme
- No ChiLin software version is stated, and no concrete commands or environment specification are provided for reproducing the atlas compilation; the documented install targets Python 2

## Evidence

- [results] read-layer cutoffs: [evidence span withheld — anchor status: not_found]
- [abstract] atlas scale and categories: "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [methods] mapping step: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [methods] PBC step: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [readme] pipeline scope: "It is a pipeline for processing ChIP-seq and DNase-seq data."
