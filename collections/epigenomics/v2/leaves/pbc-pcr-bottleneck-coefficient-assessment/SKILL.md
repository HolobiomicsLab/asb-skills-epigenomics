---
name: pbc-pcr-bottleneck-coefficient-assessment
description: Use when QC'ing ChIP-seq or DNase-seq libraries for sequencing-depth
  bottlenecking — e.g., checking whether a sample's PBC falls above the atlas-derived
  0.8 cutoff or comparing it to historical read-layer metric distributions.
when_to_use_negative:
- Input data are already peak calls or signal tracks rather than aligned BAM files
  — PBC requires read-level duplication statistics
- Evaluating peak-level QC such as FRiP scores or replicate peak overlap/correlation,
  which are ChiLin steps 5-7, not the PBC read-layer assessment
- Data types outside ChIP-seq/DNase-seq where the 23,677-sample atlas cutoffs (PBC
  > 0.8, unique ratio > 0.5, 25M/40M median depth) may not apply
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: ChiLin
  role: pipeline executing step 4 PBC evaluation on subsampled BAM files and emitting
    per-sample QC JSON
  repo: https://github.com/cfce/chilin
- name: bwa
  role: default mapper (step 2) producing the BAM files on which PBC is computed
  repo: https://github.com/lh3/bwa
- name: samtools
  role: BAM manipulation supporting subsampling and duplication statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:fe74ffb1f3d1985beb18a8973a4c71a9b76025d6bf9d9152a79038e2940c50e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - R
  - bwa
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pbc-pcr-bottleneck-coefficient-assessment

## Summary

Computes PCR Bottleneck Coefficient (PBC) scores on subsampled BAM files (ChiLin pipeline step 4) to quantify library complexity, and benchmarks new samples against a reference atlas of 23,677 public ChIP-seq/DNase-seq samples where >85% pass a PBC > 0.8 cutoff.

## When to use

Use when QC'ing ChIP-seq or DNase-seq libraries for sequencing-depth bottlenecking — e.g., checking whether a sample's PBC falls above the atlas-derived 0.8 cutoff or comparing it to historical read-layer metric distributions.

## When NOT to use

- Input data are already peak calls or signal tracks rather than aligned BAM files — PBC requires read-level duplication statistics
- Evaluating peak-level QC such as FRiP scores or replicate peak overlap/correlation, which are ChiLin steps 5-7, not the PBC read-layer assessment
- Data types outside ChIP-seq/DNase-seq where the 23,677-sample atlas cutoffs (PBC > 0.8, unique ratio > 0.5, 25M/40M median depth) may not apply

## Inputs

- BAM files from bwa (default), bowtie, or star mapping (ChiLin step 2)
- Subsampled BAM files (shared with step 3 macs2 fragment-size estimation)
- Reference genome index (e.g., hg19/hg38/mm9/mm10 tgz downloads from cistrome.org)

## Outputs

- Per-sample id_pbc. JSON record with PBC score
- Aggregated category-level cutoff and median-read-depth table (qc_metric_summary.csv)
- Historical-vs-new QC comparison plot (id_bwa_compare.R -> id_bwa_compare.pdf)

## How to apply

Run ChiLin step 4 (PBC evaluation) on subsampled BAM files produced by bwa (default) mapping in step 2; if step 3 (macs2 fragment-size estimation on subsampled BAMs) was run, step 4 reuses those subsampled files. Parse the per-sample id_pbc. JSON output together with id_fastqc. and id_map. records to assemble read-layer metrics. Assign samples to their literature-based category and compute the cumulative fraction passing PBC > 0.8 alongside quality score > 25 and unique mapped ratio > 0.5 — the paper reports >85% of the 23,677 public samples pass each of these. Render new-sample-versus-historical comparisons with the id_bwa_compare R script (id_bwa_compare.R -> id_bwa_compare.pdf) and flag any category deviating from the reported cumulative cutoffs.

## Related tools

- **ChiLin** (pipeline executing step 4 PBC evaluation on subsampled BAM files and emitting per-sample QC JSON) — https://github.com/cfce/chilin
- **bwa** (default mapper (step 2) producing the BAM files on which PBC is computed) — https://github.com/lh3/bwa
- **samtools** (BAM manipulation supporting subsampling and duplication statistics)

## Examples

```
conda create -n chilin_env python=2 && conda install bwa=0.7.13 samtools=0.1.19 ... macs2=2.1.0 && python setup.py install && chilin -h
```

## Evaluation signals

- >85% of samples in a category should show PBC > 0.8; a lower cumulative fraction flags category-level deviation
- Per-sample id_pbc. JSON records must be present and parseable for every sample processed
- Samples should be assigned to one of the eight literature-based categories covering the 23,677 public samples (11,265 datasets)
- Median read depths in the summary table should be ~25M reads for ChIP-seq and ~40M for DNase-seq
- The id_bwa_compare.pdf comparison output should render without error against the historical QC distribution

## Limitations

- PBC evaluation uses subsampled BAM files and is skipped if step 3 fragment-size estimation was run (files are reused), so subsampling choices propagate between steps
- No software version is stated for ChiLin, and reproducibility is claimed without concrete commands, environment specifications, or output artifacts in the discussion
- The atlas cutoffs derive from a specific 2016 compilation of public GEO samples; newer library preparations or other assay types may shift the PBC distribution

## Evidence

- [methods] PBC evaluation step on subsampled BAMs: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [abstract] Atlas-derived cutoff finding: [evidence span withheld — anchor status: not_found]
- [abstract] Dataset scale and categories: "over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [other] Comparison rendering via R script: [evidence span withheld — anchor status: not_found]
- [readme] ChiLin environment install from README: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
