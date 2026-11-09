---
name: insert-size-metrics-profiling
description: Use when processing paired-end cfMeDIP-seq (or cfDNA) BAM alignments
  within a Snakemake QC pipeline and you need fragment-size distributions and windowed
  short/long fragmentation ratios as quality-control metrics.
when_to_use_negative:
- Data are single-end — MEDIPIPE runs Picard CollectInsertSizeMetrics and computes
  fragmentation size ratios for paired-end data only
- Alignment stage is incomplete — the BAM must already be filtered and duplicate-removed;
  running insert-size metrics on unfiltered or un-deduplicated alignments produces
  inflated fragment-size estimates
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Picard
  role: CollectInsertSizeMetrics estimates fragment sizes from the deduplicated paired-end
    BAM; ratios are computed in 1 Mb and 5 Mb windows
- name: SAMtools
  role: Filters alignments (unmapped, secondary, improperly paired) upstream and provides
    per-sample BAM statistics plus quickcheck validation of the sorted BAM
- name: BWA-MEM
  role: Maps Trim Galore-processed reads to the spike-in-aware reference to produce
    the alignment input
- name: UMI-tools
  role: Performs deduplication when UMI barcodes were added, producing the BAM on
    which insert-size metrics are computed
- name: Snakemake
  role: Wraps the insert-size and fragmentation-ratio steps with containerized execution
    environments and YAML-driven configuration (spike-in declarations, UMI presence)
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:4559439d659e70262bcd1050757996156e2bf044185fc6ae4ed8536b65976d52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - BWA-MEM
  - Picard
  - SAMtools
  - Snakemake
  - UMI-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# insert-size-metrics-profiling

## Summary

Estimate cfDNA fragment-size distributions from paired-end cfMeDIP-seq alignments using Picard CollectInsertSizeMetrics, and derive short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb genomic windows (Cristiano et al. 2019) as MEDIPIPE QC metrics.

## When to use

Use when processing paired-end cfMeDIP-seq (or cfDNA) BAM alignments within a Snakemake QC pipeline and you need fragment-size distributions and windowed short/long fragmentation ratios as quality-control metrics.

## When NOT to use

- Data are single-end — MEDIPIPE runs Picard CollectInsertSizeMetrics and computes fragmentation size ratios for paired-end data only
- Alignment stage is incomplete — the BAM must already be filtered and duplicate-removed; running insert-size metrics on unfiltered or un-deduplicated alignments produces inflated fragment-size estimates

## Inputs

- filtered, duplicate-removed, sorted paired-end BAM file (SAMtools markdup or UMI-tools dedup output)
- primary genome reference with spike-in sequences appended as declared in the YAML configuration file

## Outputs

- Picard CollectInsertSizeMetrics insert-size metrics report
- fragment-size-ratio table with window columns for the 1 Mb and 5 Mb resolutions
- per-sample SAMtools BAM statistics used as QC metrics for downstream aggregation

## How to apply

Only run this step on paired-end data, after BWA-MEM mapping to the spike-in-aware reference, SAMtools filtering (discarding unmapped, secondary, and improperly paired reads), and duplicate removal (SAMtools markdup for non-UMI libraries or UMI-tools dedup when UMI barcodes were extracted upstream). Run Picard CollectInsertSizeMetrics on the deduplicated BAM to estimate fragment sizes, then compute short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb windows following Cristiano et al. 2019. Additionally collect per-sample BAM statistics via SAMtools so the insert-size results can be aggregated with other QC metrics. Validate that the deduplicated BAM passes SAMtools quickcheck as a sorted BAM and that the insert-size metrics report and fragment-size-ratio table are non-empty with columns for both the 1 Mb and 5 Mb window resolutions.

## Related tools

- **Picard** (CollectInsertSizeMetrics estimates fragment sizes from the deduplicated paired-end BAM; ratios are computed in 1 Mb and 5 Mb windows)
- **SAMtools** (Filters alignments (unmapped, secondary, improperly paired) upstream and provides per-sample BAM statistics plus quickcheck validation of the sorted BAM)
- **BWA-MEM** (Maps Trim Galore-processed reads to the spike-in-aware reference to produce the alignment input)
- **UMI-tools** (Performs deduplication when UMI barcodes were added, producing the BAM on which insert-size metrics are computed)
- **Snakemake** (Wraps the insert-size and fragmentation-ratio steps with containerized execution environments and YAML-driven configuration (spike-in declarations, UMI presence)) — https://github.com/pughlab/MEDIPIPE

## Evaluation signals

- The filtered, duplicate-removed BAM file exists and passes SAMtools quickcheck as a valid sorted BAM
- The insert-size metrics report exists and is non-empty
- The fragment-size-ratio table is non-empty and contains window columns for both the 1 Mb and 5 Mb resolutions
- Per-sample SAMtools BAM statistics were collected and are available for downstream QC aggregation

## Limitations

- Fragment-size estimation and fragmentation-ratio computation are restricted to paired-end libraries; single-end cfMeDIP-seq data cannot use this metric
- Results depend on correct UMI configuration: using SAMtools markdup on UMI-bearing libraries (or UMI-tools dedup on non-UMI libraries) distorts duplicate removal and hence insert-size metrics
- Spike-in handling must be declared in the YAML configuration; aligned spike-in reads are extracted separately for quantity assessment rather than entering the insert-size step

## Evidence

- [intro] paired-end scope and flexibility: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
- [readme] pipeline scope from raw FASTQ to QC aggregation: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [readme] Snakemake deployment context for the QC steps: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments."
