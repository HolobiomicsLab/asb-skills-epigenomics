---
name: sam-alignment-filtering-and-duplicate-removal
description: Use when processing cfMeDIP-seq (or MeDIP-seq) FASTQ reads into filtered,
  deduplicated BAM files with QC metrics, especially when libraries may or may not
  carry UMI barcodes and spike-in controls may be present.
when_to_use_negative:
- Input data is not cfMeDIP-seq/MeDIP-seq FASTQ (e.g. whole-genome bisulfite reads)
  — the spike-in-aware and methylation-oriented QC design would not apply
- Reads have not been adapter/quality trimmed — Trim Galore-processed input is assumed
  by the alignment step
- The task is downstream methylation quantification or multi-sample aggregation rather
  than per-sample alignment and QC
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Maps Trim Galore-processed reads to the spike-in-aware primary genome reference
  repo: https://github.com/pughlab/MEDIPIPE
- name: SAMtools
  role: Filters alignments (unmapped, secondary, improperly paired) and provides markdup
    for non-UMI libraries plus per-sample BAM statistics
  repo: https://github.com/pughlab/MEDIPIPE
- name: UMI-tools
  role: Duplicate removal via dedup when UMI barcodes were added for sequence error
    suppression
  repo: https://github.com/pughlab/MEDIPIPE
- name: Picard
  role: CollectInsertSizeMetrics for fragment-size estimation on paired-end data
  repo: https://github.com/pughlab/MEDIPIPE
- name: Snakemake
  role: Orchestrates the module steps with containerized execution environments deployed
    via Conda
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:8135087af23114727ff56def0ad405af2b8bc310a0cd1bed110d822ea60caf7c
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
n_claims: 10
---

# sam-alignment-filtering-and-duplicate-removal

## Summary

Reconstructs MEDIPIPE's alignment and QC module for cfMeDIP-seq: BWA-MEM mapping against a spike-in-aware reference, SAMtools filtering, conditional duplicate removal (SAMtools markdup vs UMI-tools dedup), Picard insert-size metrics, and SAMtools BAM statistics.

## When to use

Use when processing cfMeDIP-seq (or MeDIP-seq) FASTQ reads into filtered, deduplicated BAM files with QC metrics, especially when libraries may or may not carry UMI barcodes and spike-in controls may be present.

## When NOT to use

- Input data is not cfMeDIP-seq/MeDIP-seq FASTQ (e.g. whole-genome bisulfite reads) — the spike-in-aware and methylation-oriented QC design would not apply
- Reads have not been adapter/quality trimmed — Trim Galore-processed input is assumed by the alignment step
- The task is downstream methylation quantification or multi-sample aggregation rather than per-sample alignment and QC

## Inputs

- Trim Galore-processed FASTQ files (single-end or paired-end)
- Primary genome reference FASTA, optionally with spike-in sequences appended
- YAML configuration file declaring spike-in controls and UMI usage
- UMI barcode information extracted upstream (when UMI libraries are used)

## Outputs

- Filtered, duplicate-removed sorted BAM file
- Aligned spike-in read quantities (when spike-ins are used)
- Picard CollectInsertSizeMetrics report (paired-end only)
- Fragment-size-ratio table with columns for 1 Mb and 5 Mb windows
- Per-sample SAMtools BAM statistics for QC aggregation

## How to apply

Map Trim Galore-processed reads to the primary genome reference — with spike-in sequences appended when spike-ins are declared in the YAML configuration — using BWA-MEM, and extract aligned spike-in reads separately when spike-ins are used. Filter alignments with SAMtools to discard unmapped reads, secondary alignments, and improperly paired mates for paired-end libraries. Dispatch duplicate removal by library preparation: SAMtools markdup when no UMI barcodes were added, or UMI-tools dedup when UMI barcodes were extracted upstream for sequence error suppression. For paired-end data only, estimate fragment sizes with Picard CollectInsertSizeMetrics and compute short-to-long fragmentation size ratios in consecutive 1 Mb and 5 Mb windows (Cristiano et al. 2019), then collect per-sample BAM statistics via SAMtools for downstream QC aggregation.

## Related tools

- **BWA-MEM** (Maps Trim Galore-processed reads to the spike-in-aware primary genome reference) — https://github.com/pughlab/MEDIPIPE
- **SAMtools** (Filters alignments (unmapped, secondary, improperly paired) and provides markdup for non-UMI libraries plus per-sample BAM statistics) — https://github.com/pughlab/MEDIPIPE
- **UMI-tools** (Duplicate removal via dedup when UMI barcodes were added for sequence error suppression) — https://github.com/pughlab/MEDIPIPE
- **Picard** (CollectInsertSizeMetrics for fragment-size estimation on paired-end data) — https://github.com/pughlab/MEDIPIPE
- **Snakemake** (Orchestrates the module steps with containerized execution environments deployed via Conda) — https://github.com/pughlab/MEDIPIPE

## Examples

```
bwa mem -t 8 reference_with_spikeins.fa sample_R1.trim.fastq.gz sample_R2.trim.fastq.gz | samtools view -b -F 1804 - | samtools sort -o sample.sorted.bam
```

## Evaluation signals

- SAMtools quickcheck passes on the filtered, duplicate-removed BAM and confirms it is a valid sorted BAM
- The insert-size metrics report is non-empty (paired-end data only)
- The fragment-size-ratio table is non-empty and contains window columns for both the 1 Mb and 5 Mb resolutions
- Per-sample BAM statistics are produced for downstream QC aggregation
- Spike-in aligned read quantities are extracted separately when spike-ins are declared in the YAML configuration

## Limitations

- Insert-size estimation and fragmentation size ratios are computed only for paired-end data
- Duplicate-removal strategy depends on library type: markdup is only correct for non-UMI libraries; using it on UMI-tagged libraries would ignore the molecular barcodes
- Spike-in handling requires spike-in sequences to be declared in the YAML configuration file; without that, spike-in QC quantities are not extracted

## Evidence

- [full_text] MEDIPIPE dispatches duplicate removal between two tools: SAMtools markdup is applied when no UMI barcodes were added to the library, whereas UMI-tools dedup is applied when UMI barcodes were added for sequence error suppression.: [evidence span withheld — anchor status: not_found]
- [full_text] Alignment against spike-in-aware reference and SAMtools filtering steps.: [evidence span withheld — anchor status: not_found]
- [full_text] Filtering criteria for alignments.: [evidence span withheld — anchor status: not_found]
- [full_text] Insert-size and fragmentation QC for paired-end data.: [evidence span withheld — anchor status: not_found]
- [full_text] Validation of module outputs.: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline scope and flexibility from README.: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
