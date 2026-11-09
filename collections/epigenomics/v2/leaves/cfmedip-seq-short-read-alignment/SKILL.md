---
name: cfmedip-seq-short-read-alignment
description: Use when raw or Trim Galore-trimmed cfMeDIP-seq FASTQs (single-end or
  paired-end, with or without spike-ins/UMIs) must be mapped, duplicate-processed,
  and QC-assessed to produce sorted BAMs and fragment-size metrics.
when_to_use_negative:
- Input is raw FASTQ that has not yet been adapter/quality trimmed with Trim Galore
  — pre-processing must happen before BWA-MEM alignment
- Data type is not cfMeDIP-seq / (cf)MeDIP-seq methylome data (e.g. whole-genome bisulfite
  or regular ChIP-seq) — MEDIPIPE is scoped to (cf)MeDIP-seq QC and analysis
- Alignment-only is needed without duplicate handling — the skill specifically dispatches
  between SAMtools markdup and UMI-tools dedup based on library UMI status
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Maps Trim Galore-processed reads to the primary genome reference (with spike-in
    sequences appended when declared)
- name: SAMtools
  role: Filters unmapped reads, secondary alignments, and improperly paired mates;
    markdup removes duplicates for non-UMI libraries; also collects per-sample BAM
    statistics
- name: UMI-tools
  role: Deduplicates reads when UMI barcodes were added to the library for sequence
    error suppression
- name: Picard
  role: CollectInsertSizeMetrics estimates fragment sizes for paired-end data used
    in short-to-long fragmentation ratio computation
- name: Snakemake
  role: Workflow engine that orchestrates the alignment and QC module with automatically
    deployed Conda execution environments
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:ed83d133f2764ba319b8686ce6b4c85137382b53be6c29804bbe92a476308f5a
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cfmedip-seq-short-read-alignment

## Summary

Align Trim Galore-processed cfMeDIP-seq reads with BWA-MEM to a primary genome reference (plus spike-in sequences when declared in the YAML config), then filter, deduplicate (SAMtools markdup vs UMI-tools dedup depending on UMI usage), and collect alignment QC with Picard insert-size metrics and SAMtools BAM statistics.

## When to use

Use when raw or Trim Galore-trimmed cfMeDIP-seq FASTQs (single-end or paired-end, with or without spike-ins/UMIs) must be mapped, duplicate-processed, and QC-assessed to produce sorted BAMs and fragment-size metrics.

## When NOT to use

- Input is raw FASTQ that has not yet been adapter/quality trimmed with Trim Galore — pre-processing must happen before BWA-MEM alignment
- Data type is not cfMeDIP-seq / (cf)MeDIP-seq methylome data (e.g. whole-genome bisulfite or regular ChIP-seq) — MEDIPIPE is scoped to (cf)MeDIP-seq QC and analysis
- Alignment-only is needed without duplicate handling — the skill specifically dispatches between SAMtools markdup and UMI-tools dedup based on library UMI status

## Inputs

- Trim Galore-processed FASTQ files (single-end or paired-end cfMeDIP-seq reads)
- Primary genome reference FASTA, optionally with spike-in sequences appended
- YAML configuration file declaring spike-in controls and UMI usage
- UMI barcode information when UMIs were extracted upstream

## Outputs

- Filtered, duplicate-removed sorted BAM file
- Extracted spike-in alignment reads for quantity assessment (when spike-ins are used)
- Picard CollectInsertSizeMetrics report (paired-end only)
- Fragment-size-ratio table with 1 Mb and 5 Mb window columns (paired-end only)
- Per-sample SAMtools BAM statistics for QC aggregation

## How to apply

Map the Trim Galore-processed reads to the primary genome reference using BWA-MEM, appending spike-in sequences to the reference when spike-in controls are declared in the YAML configuration file; if spike-ins are used, extract aligned spike-in reads for separate quantity assessment. Filter alignments with SAMtools to discard unmapped reads, secondary alignments, and improperly paired mates (paired-end libraries). Choose the deduplication route by library construction: SAMtools markdup when no UMI barcodes were added, or UMI-tools dedup when UMI barcodes were extracted upstream for sequence error suppression. For paired-end data only, run Picard CollectInsertSizeMetrics and compute short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb windows (Cristiano et al. 2019), then collect per-sample BAM statistics via SAMtools as QC metrics for downstream aggregation.

## Related tools

- **BWA-MEM** (Maps Trim Galore-processed reads to the primary genome reference (with spike-in sequences appended when declared))
- **SAMtools** (Filters unmapped reads, secondary alignments, and improperly paired mates; markdup removes duplicates for non-UMI libraries; also collects per-sample BAM statistics)
- **UMI-tools** (Deduplicates reads when UMI barcodes were added to the library for sequence error suppression)
- **Picard** (CollectInsertSizeMetrics estimates fragment sizes for paired-end data used in short-to-long fragmentation ratio computation)
- **Snakemake** (Workflow engine that orchestrates the alignment and QC module with automatically deployed Conda execution environments) — https://github.com/pughlab/MEDIPIPE

## Examples

```
snakemake --use-conda --cores 8 --configfile config.yaml
```

## Evaluation signals

- SAMtools quickcheck passes on the filtered, duplicate-removed BAM (valid sorted BAM)
- Picard insert-size metrics report exists and is non-empty (paired-end data only)
- Fragment-size-ratio table is non-empty and contains window columns for both the 1 Mb and 5 Mb resolutions
- Per-sample SAMtools BAM statistics files are produced for downstream QC aggregation
- Spike-in reads were extracted separately when spike-ins were declared in the YAML configuration

## Limitations

- Insert-size estimation and fragmentation-ratio computation apply only to paired-end data
- The deduplication tool choice is conditional: SAMtools markdup cannot account for UMI molecular barcodes, and UMI-tools dedup is only applicable when UMIs were extracted upstream
- Spike-in-aware alignment requires spike-in sequences to be declared in the YAML configuration file; otherwise the reference contains only the primary genome

## Evidence

- [full_text] MEDIPIPE dispatches duplicate removal between two tools: [evidence span withheld — anchor status: not_found]
- [full_text] alignment workflow: [evidence span withheld — anchor status: not_found]
- [full_text] filter steps: [evidence span withheld — anchor status: not_found]
- [full_text] insert-size QC: [evidence span withheld — anchor status: not_found]
- [readme] pipeline scope: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
