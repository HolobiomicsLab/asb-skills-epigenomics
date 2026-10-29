---
name: fragment-size-ratio-computation
description: Use when processing paired-end cfMeDIP-seq BAMs within MEDIPIPE's alignment/QC
  module and you need insert-size distributions and fragmentation-ratio QC tables
  from duplicate-removed alignments.
when_to_use_negative:
- Input library is single-end — insert-size estimation and short-to-long ratio computation
  are performed for paired-end data only
- Alignment and duplicate removal have not yet been performed (no filtered, duplicate-removed
  sorted BAM exists)
- Task is methylation quantification or sample aggregation rather than alignment/QC
  metrics generation
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Picard
  role: CollectInsertSizeMetrics estimates fragment sizes from paired-end BAMs
- name: SAMtools
  role: Filters alignments (unmapped, secondary, improperly paired mates), supports
    markdup deduplication, collects BAM statistics, and validates BAMs via quickcheck
- name: UMI-tools
  role: Deduplicates reads when UMI barcodes were added for sequence error suppression,
    producing the BAM on which insert-size metrics are computed
- name: BWA-MEM
  role: Maps Trim Galore-processed reads to the primary genome reference with spike-in
    sequences appended
- name: Snakemake
  role: Orchestrates the alignment/QC module and its environment deployment
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:570620f307268e7ab86450b29edf5c4c324119a760a24e77a1c07d1132cb826a
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# fragment-size-ratio-computation

## Summary

A cfMeDIP-seq QC skill from MEDIPIPE that estimates fragment sizes with Picard CollectInsertSizeMetrics and computes short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb windows (Cristiano et al. 2019) as cell-free DNA fragmentation-quality metrics.

## When to use

Use when processing paired-end cfMeDIP-seq BAMs within MEDIPIPE's alignment/QC module and you need insert-size distributions and fragmentation-ratio QC tables from duplicate-removed alignments.

## When NOT to use

- Input library is single-end — insert-size estimation and short-to-long ratio computation are performed for paired-end data only
- Alignment and duplicate removal have not yet been performed (no filtered, duplicate-removed sorted BAM exists)
- Task is methylation quantification or sample aggregation rather than alignment/QC metrics generation

## Inputs

- Filtered, duplicate-removed, sorted BAM from paired-end cfMeDIP-seq libraries (post BWA-MEM alignment and SAMtools filtering/deduplication)
- YAML configuration file declaring paired-end status, spike-in controls, and UMI usage
- Genome window definitions for the 1 Mb and 5 Mb binning resolutions

## Outputs

- Picard CollectInsertSizeMetrics insert-size metrics report
- Fragment-size-ratio table with window columns for the 1 Mb and 5 Mb resolutions
- Per-sample BAM statistics from SAMtools for downstream QC aggregation

## How to apply

After mapping Trim Galore-processed reads with BWA-MEM to the primary genome reference (with spike-in sequences appended when declared in the YAML configuration) and filtering with SAMtools to discard unmapped reads, secondary alignments, and improperly paired mates, remove duplicates with SAMtools markdup (non-UMI libraries) or UMI-tools dedup (UMI barcodes extracted upstream). Then, for paired-end data only, run Picard CollectInsertSizeMetrics on the filtered, duplicate-removed BAM to estimate fragment sizes. Compute short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb genomic windows following Cristiano et al. 2019. Collect per-sample BAM statistics via SAMtools so these metrics can be aggregated downstream; the ratio computation is deliberately skipped for single-end libraries because fragment sizes cannot be inferred without paired mates.

## Related tools

- **Picard** (CollectInsertSizeMetrics estimates fragment sizes from paired-end BAMs)
- **SAMtools** (Filters alignments (unmapped, secondary, improperly paired mates), supports markdup deduplication, collects BAM statistics, and validates BAMs via quickcheck)
- **UMI-tools** (Deduplicates reads when UMI barcodes were added for sequence error suppression, producing the BAM on which insert-size metrics are computed)
- **BWA-MEM** (Maps Trim Galore-processed reads to the primary genome reference with spike-in sequences appended)
- **Snakemake** (Orchestrates the alignment/QC module and its environment deployment) — https://github.com/pughlab/MEDIPIPE

## Examples

```
picard CollectInsertSizeMetrics I=sample.dedup.bam O=insert_size_metrics.txt H=insert_size_histogram.pdf
```

## Evaluation signals

- The filtered, duplicate-removed BAM file exists and passes SAMtools quickcheck as a valid sorted BAM
- The insert-size metrics report is non-empty
- The fragment-size-ratio table is non-empty and contains window columns for both the 1 Mb and 5 Mb resolutions
- Ratios and insert-size metrics are absent/skipped for single-end samples but present for all paired-end samples
- Per-sample BAM statistics are collected via SAMtools and available for downstream aggregation

## Limitations

- Fragment-size estimation and ratio computation apply to paired-end data only; single-end inputs cannot yield insert-size distributions
- Duplicate-removal tool choice depends on library prep: SAMtools markdup for non-UMI libraries versus UMI-tools dedup when UMI barcodes were extracted upstream — using the wrong path changes the input BAM
- Spike-in-aware alignment requires spike-in sequences to be declared in the YAML configuration file for reference appending and spike-in read extraction

## Evidence

- [methods] For paired-end data only, estimate fragment sizes with Picard CollectInsertSizeMetrics and compute short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb windows: [evidence span withheld — anchor status: not_found]
- [methods] Validation requires the filtered BAM and non-empty QC reports: [evidence span withheld — anchor status: not_found]
- [readme] MEDIPIPE handles single- or paired-end reads with or without spike-in/UMI sequences: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences"
