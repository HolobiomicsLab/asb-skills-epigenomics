---
name: historical-qc-atlas-benchmark-comparison
description: Use when a new ChIP-seq or DNase-seq sample needs QC assessment against
  historical public data, or when validating whether read-layer metrics (FastQC score,
  mapped ratio, PBC) are assay-type independent.
when_to_use_negative:
- The analysis goal is peak-level or peak-layer QC (FRiP score, replicate peak overlap/correlation,
  fragment size estimation) rather than read-layer metrics.
- Input data is not ChIP-seq or DNase-seq (e.g. RNA-seq or ATAC-seq), since the atlas
  cutoffs were derived specifically from ChIP-seq/DNase-seq samples.
- Only a handful of samples exist, so cumulative distribution statistics over the
  eight categories cannot be meaningfully estimated.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3574
tools:
- name: ChiLin
  role: Unified pipeline executing the read-layer QC steps (FastQC evaluation, bwa
    mapping, PBC evaluation) and generating QC reports with comparisons against historical
    data
  repo: https://github.com/cfce/chilin
- name: FastQC
  role: 'Step 1: sequence quality score evaluation feeding the >25 quality cutoff
    distribution'
- name: bwa
  role: Step 2 default mapper producing the uniquely mapped read ratios compared against
    the 0.5 cutoff
  repo: https://github.com/lh3/bwa
- name: samtools
  role: Supports BAM processing for mapping statistics and PBC computation on subsampled
    BAM files
- name: R
  role: Runs id_bwa_compare.R to render the new-sample-versus-historical-distribution
    comparison plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:c36f498ac1691ef46983e1fa23f56c8aa3c64508105c65c71d256088eeff608d
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# historical-qc-atlas-benchmark-comparison

## Summary

Reproduces ChiLin's read-layer QC quality atlas by computing cumulative distributions of FastQC sequence quality scores, uniquely mapped read ratios (bwa), and PBC values over thousands of public ChIP-seq/DNase-seq samples, deriving assay-type-insensitive cutoffs (quality score >25, unique ratio >0.5, PBC >0.8), and benchmarking new samples against these historical distributions.

## When to use

Use when a new ChIP-seq or DNase-seq sample needs QC assessment against historical public data, or when validating whether read-layer metrics (FastQC score, mapped ratio, PBC) are assay-type independent.

## When NOT to use

- The analysis goal is peak-level or peak-layer QC (FRiP score, replicate peak overlap/correlation, fragment size estimation) rather than read-layer metrics.
- Input data is not ChIP-seq or DNase-seq (e.g. RNA-seq or ATAC-seq), since the atlas cutoffs were derived specifically from ChIP-seq/DNase-seq samples.
- Only a handful of samples exist, so cumulative distribution statistics over the eight categories cannot be meaningfully estimated.

## Inputs

- FASTQ read files from ChIP-seq or DNase-seq samples
- Per-sample ChiLin QC JSON records (id_fastqc., id_map., id_pbc.)
- Historical QC atlas of 23,677 public ChIP-seq/DNase-seq samples (11,265 datasets) across eight literature-based categories
- id_bwa_compare.R comparison script

## Outputs

- qc_metric_summary.csv with category-level cumulative cutoffs and median read depths
- id_bwa_compare.pdf comparing a new sample's mapping statistics against the historical QC distribution
- Per-sample FastQC quality scores, uniquely mapped ratios, and PBC values

## How to apply

Run the ChiLin pipeline read-layer QC steps on each sample: step 1 FastQC sequence quality evaluation, step 2 bwa (default) mapping to produce the BAM, and step 4 PBC evaluation on subsampled BAM files, yielding id_fastqc., id_map., and id_pbc. records per sample. Parse the per-sample QC JSON records with Python and assign samples to the eight literature-based categories of the 23,677-sample (11,265 dataset) atlas. Compute per-category cumulative distributions and cutoff pass fractions, expecting >85% of samples with quality score >25, uniquely mapped ratio >0.5, and PBC >0.8, plus median read depths of 25M (ChIP-seq) and 40M (DNase-seq). Write the aggregated cutoff/depth table (e.g. qc_metric_summary.csv) and render new-sample comparisons against the historical distribution with the id_bwa_compare R script (id_bwa_compare.R -> id_bwa_compare.pdf). Flag any category whose computed distributions deviate from the reported cumulative cutoffs.

## Related tools

- **ChiLin** (Unified pipeline executing the read-layer QC steps (FastQC evaluation, bwa mapping, PBC evaluation) and generating QC reports with comparisons against historical data) — https://github.com/cfce/chilin
- **FastQC** (Step 1: sequence quality score evaluation feeding the >25 quality cutoff distribution)
- **bwa** (Step 2 default mapper producing the uniquely mapped read ratios compared against the 0.5 cutoff) — https://github.com/lh3/bwa
- **samtools** (Supports BAM processing for mapping statistics and PBC computation on subsampled BAM files)
- **R** (Runs id_bwa_compare.R to render the new-sample-versus-historical-distribution comparison plot)

## Examples

```
git clone https://github.com/cfce/chilin && cd chilin && bash demo/docker_test.sh
```

## Evaluation signals

- More than 85% of samples in the compiled distributions have sequence quality score >25
- More than 85% of samples have uniquely mapped read ratio >0.5 and PBC >0.8
- Median read depth reproduces 25M reads for ChIP-seq and 40M for DNase-seq categories
- qc_metric_summary.csv covers all eight literature-based categories with per-category cumulative fractions
- id_bwa_compare.pdf is produced and places the new sample within the historical mapping-ratio distribution

## Limitations

- No software version is stated for ChiLin, complicating exact reproduction; the reproducibility claim is not backed by concrete environment specifications in all sections.
- Cutoffs were derived from 23,677 public GEO samples under eight literature-based categories and hg19/hg38/mm9/mm10 references; samples outside these assay classes may not satisfy the same thresholds.
- PBC and meta-region read ratios are computed on subsampled BAM files, so values depend on the subsampling step.

## Evidence

- [abstract] quality atlas of 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets): "comparisons with historical data derived from over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [results] >85% cutoffs for FastQC, mapped ratio, PBC: [evidence span withheld — anchor status: not_found]
- [methods] pipeline step definitions: "step 1(*can skip*): FastQC sequence quality evaluation, reads GC contents evaluation and library contamination"
- [methods] mapping step: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [methods] PBC evaluation on subsampled BAM: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [abstract] QC report comparison against historical data: "ChiLin generates comprehensive quality control reports that include comparisons with historical data"
- [readme] docker demo invocation: "run the demo command
sh
bash demo/docker_test.sh
"
