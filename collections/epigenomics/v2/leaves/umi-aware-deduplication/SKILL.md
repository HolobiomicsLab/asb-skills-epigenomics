---
name: umi-aware-deduplication
description: Decision support for choosing between SAMtools markdup and UMI-tools
  dedup when aligning cfMeDIP-seq libraries that may or may not incorporate UMI barcodes,
  as declared in the YAML configuration.
when_to_use_negative:
- When processing raw FASTQ files that have not yet been adapter-trimmed by Trim Galore
  and aligned with BWA-MEM — deduplication must follow the filtering step
- When the data type is not (cf)MeDIP-seq; MEDIPIPE is designed specifically for cfMeDIP-seq
  QC and analysis
- When UMI barcodes are present but were not extracted upstream — UMI-tools dedup
  requires UMI information and coordinate-only markdup would be the mismatched alternative
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BWA-MEM
  role: Maps Trim Galore-processed reads to the primary genome reference with spike-in
    sequences appended
  repo: https://github.com/pughlab/MEDIPIPE
- name: SAMtools
  role: Filters alignments (unmapped, secondary, improperly-paired reads) and performs
    coordinate-based markdup for non-UMI libraries; also collects BAM statistics
  repo: https://github.com/pughlab/MEDIPIPE
- name: UMI-tools
  role: Removes duplicate reads via UMI-barcode-aware dedup when UMI barcodes were
    extracted upstream
  repo: https://github.com/pughlab/MEDIPIPE
- name: Picard
  role: CollectInsertSizeMetrics estimates fragment sizes for paired-end data used
    in fragmentation-ratio QC
  repo: https://github.com/pughlab/MEDIPIPE
- name: Snakemake
  role: Orchestrates the conditional dedup workflow with automatically deployed execution
    environments
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:600eec5201ced1ba7ae17759e9e62e1d6ca2f70fe18c2f4af326ee0dc55c601e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# umi-aware-deduplication

## Summary

A conditional duplicate-removal step within MEDIPIPE's alignment/QC module for cfMeDIP-seq data: SAMtools markdup is used for libraries without UMI barcodes, while UMI-tools dedup is used when UMI barcodes were extracted upstream for sequence error suppression.

## When to use

Decision support for choosing between SAMtools markdup and UMI-tools dedup when aligning cfMeDIP-seq libraries that may or may not incorporate UMI barcodes, as declared in the YAML configuration.

## When NOT to use

- When processing raw FASTQ files that have not yet been adapter-trimmed by Trim Galore and aligned with BWA-MEM — deduplication must follow the filtering step
- When the data type is not (cf)MeDIP-seq; MEDIPIPE is designed specifically for cfMeDIP-seq QC and analysis
- When UMI barcodes are present but were not extracted upstream — UMI-tools dedup requires UMI information and coordinate-only markdup would be the mismatched alternative

## Inputs

- Trim Galore-processed FASTQ reads (single-end or paired-end)
- BWA-MEM alignments filtered with SAMtools (unmapped, secondary, and improperly-paired reads removed)
- UMI barcode information extracted upstream (when UMI protocols are used)
- YAML configuration file declaring spike-in controls and UMI usage
- Genome reference with spike-in sequences appended (when spike-ins are declared)

## Outputs

- Filtered, duplicate-removed sorted BAM file
- Extracted spike-in read alignments for quantity assessment
- Picard CollectInsertSizeMetrics report (paired-end only)
- Fragment-size-ratio table with 1 Mb and 5 Mb window columns (paired-end only)
- Per-sample BAM statistics (SAMtools) for downstream QC aggregation

## How to apply

First map Trim Galore-processed reads to the primary genome reference using BWA-MEM, appending spike-in sequences to the reference when spike-in controls are declared in the YAML configuration file, and extract aligned spike-in reads for separate quantity assessment. Filter alignments with SAMtools to discard unmapped reads, secondary alignments, and improperly paired mates for paired-end libraries. Then dispatch duplicate removal by library type: apply SAMtools markdup when no UMI barcodes were added, or UMI-tools dedup when UMI barcodes were extracted upstream — the rationale is that UMI-based collapsing removes PCR duplicates by barcode consensus rather than by coordinate alone, so coordinate-based markdup would be inappropriate for UMI libraries. For paired-end data only, follow with Picard CollectInsertSizeMetrics and compute short-to-long fragmentation size ratios in 1 Mb and 5 Mb windows (Cristiano et al. 2019), then collect per-sample BAM statistics via SAMtools for QC aggregation.

## Related tools

- **BWA-MEM** (Maps Trim Galore-processed reads to the primary genome reference with spike-in sequences appended) — https://github.com/pughlab/MEDIPIPE
- **SAMtools** (Filters alignments (unmapped, secondary, improperly-paired reads) and performs coordinate-based markdup for non-UMI libraries; also collects BAM statistics) — https://github.com/pughlab/MEDIPIPE
- **UMI-tools** (Removes duplicate reads via UMI-barcode-aware dedup when UMI barcodes were extracted upstream) — https://github.com/pughlab/MEDIPIPE
- **Picard** (CollectInsertSizeMetrics estimates fragment sizes for paired-end data used in fragmentation-ratio QC) — https://github.com/pughlab/MEDIPIPE
- **Snakemake** (Orchestrates the conditional dedup workflow with automatically deployed execution environments) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- The filtered, duplicate-removed BAM file exists and passes SAMtools quickcheck as a valid sorted BAM
- The Picard insert-size metrics report is non-empty (paired-end libraries only)
- The fragment-size-ratio table is non-empty and contains window columns for both the 1 Mb and 5 Mb resolutions
- The correct dedup tool was dispatched: markdup only for non-UMI libraries, UMI-tools dedup only when UMI barcodes were extracted upstream
- Aligned spike-in reads were separately extracted when spike-in controls were declared in the YAML configuration

## Limitations

- Insert-size metrics and fragmentation-ratio calculations are only produced for paired-end data; single-end libraries skip this QC output
- The dedup strategy is conditional — applying coordinate-based markdup to UMI-containing libraries or UMI-tools dedup without extracted UMI barcodes would be incorrect
- Spike-in handling requires that spike-in sequences be explicitly declared in the YAML configuration file; otherwise the reference contains no spike-in sequences to extract

## Evidence

- [full_text] MEDIPIPE dispatches duplicate removal between two tools: [evidence span withheld — anchor status: not_found]
- [intro] MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences.: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
- [full_text] Filter the alignments with SAMtools to discard unmapped reads, secondary alignments, and improperly paired mates: [evidence span withheld — anchor status: not_found]
- [full_text] verify that the filtered, duplicate-removed BAM file exists and is a valid sorted BAM (SAMtools quickcheck): [evidence span withheld — anchor status: not_found]
- [readme] MEDIPIPE was developed with Snakemake, which will automatically deploy the execution environments: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments."
