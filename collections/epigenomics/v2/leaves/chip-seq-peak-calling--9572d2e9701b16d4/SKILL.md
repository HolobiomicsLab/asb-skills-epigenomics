---
name: chip-seq-peak-calling
description: Use when a ChIP-seq FASTQ dataset (with a matched input/control) needs
  peaks, peak annotation, or motif identification, and CIPHER is being used to run
  the full two-stage pipeline from raw reads to called peaks.
when_to_use_negative:
- The dataset is RNA-seq, GRO-seq, MNase-seq, DNase-seq or ATAC-seq — dispatch to
  the corresponding --mode instead of chip
- Only a subset of tools is desired (e.g. peak calling alone on existing BAMs) — CIPHER
  currently only runs entire workflows, not individual stages
- Single- and pair-ended data would need to be mixed in one config file — the README
  states CIPHER does not handle this use-case yet
edam_operation: http://edamontology.org/operation_0225
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0203
tools:
- name: CIPHER
  role: Workflow platform that dispatches --mode chip and orchestrates the two-stage
    ChIP-seq pipeline
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language in which cipher.nf is written and executed
  repo: https://www.nextflow.io/
- name: MACS2
  role: Narrow/broad peak calling on stage-1 aligned reads using the matched input
    control
- name: HOMER
  role: Peak annotation and motif identification on the called peak sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:9b8637d1d28806f07d326d19f2fed9a080fb888802fe0b6ab7ea98ceec894570
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - BBDuk
  - BBMap
  - CIPHER
  - Docker
  - FastQC
  - HOMER
  - MACS2
  - Nextflow
  edam_operation: http://edamontology.org/operation_0225
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# chip-seq-peak-calling

## Summary

CIPHER's stage-2 ChIP-seq downstream analysis that calls narrow and broad peaks on aligned reads (MACS2 with a matched input control), annotates peaks and identifies motifs (HOMER), all driven from a Nextflow workflow in a containerized environment.

## When to use

Use when a ChIP-seq FASTQ dataset (with a matched input/control) needs peaks, peak annotation, or motif identification, and CIPHER is being used to run the full two-stage pipeline from raw reads to called peaks.

## When NOT to use

- The dataset is RNA-seq, GRO-seq, MNase-seq, DNase-seq or ATAC-seq — dispatch to the corresponding --mode instead of chip
- Only a subset of tools is desired (e.g. peak calling alone on existing BAMs) — CIPHER currently only runs entire workflows, not individual stages
- Single- and pair-ended data would need to be mixed in one config file — the README states CIPHER does not handle this use-case yet

## Inputs

- Single- or pair-ended ChIP-seq FASTQ files (gzipped)
- Matched input/control FASTQ files flagged with Mark=input in the config
- Tab-separated CIPHER config file (5 or 6 columns)
- Reference genome FASTA (--fa)
- Gene annotation GTF (--gtf)

## Outputs

- Narrow and broad peak calls (MACS2 output, e.g. narrowPeak/broadPeak)
- HOMER peak annotations and identified motifs
- Sorted BAM files with alignment statistics
- Gzipped trimmed FASTQ files
- QC reports (PDF/HTML) aggregated by MultiQC in './report' or --outdir

## How to apply

Compose a tab-separated config file (5 columns single-end, 6 for pair-end: MergeID, SampleID, FASTQ path(s), InputID, Mark with 'input' for controls) so samples are paired to their input controls via InputID; do not mix single- and pair-ended rows. Launch the workflow with --mode chip, supplying --fa (FASTA), --gtf, --lib and --readLen; CIPHER's fixed stage-1 branch runs FastQC, trims adapters and filters low-quality/short reads with BBDuk (defaults mapq < 20, length < 10), and maps reads with the default BBMap aligner (alternatives bwa, bowtie2 via --aligner). The stage-2 branch then performs narrow and broad peak calling — per the source card, MACS2 with --qvalue 0.01 and --keep-dup all — followed by HOMER-based peak annotation and motif identification, with QC aggregated (e.g. deeptools plotFingerprint) by MultiQC. Results are written to './report' (override with --outdir). Because samples and inputs are paired per InputID, ensure every sample row has a corresponding input row or a '-' only where no input exists.

## Related tools

- **CIPHER** (Workflow platform that dispatches --mode chip and orchestrates the two-stage ChIP-seq pipeline) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language in which cipher.nf is written and executed) — https://www.nextflow.io/
- **MACS2** (Narrow/broad peak calling on stage-1 aligned reads using the matched input control)
- **HOMER** (Peak annotation and motif identification on the called peak sets)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.txt --fa ref.fa --gtf ref.gtf --lib s --readLen 75
```

## Evaluation signals

- Run completes and './report' (or --outdir) contains gzipped trimmed FASTQs, sorted BAMs with alignment statistics, and QC reports in PDF/HTML
- Peak files are produced for each sample with a matched InputID control, and motif/annotation output from HOMER accompanies the peak sets
- Benchmark expectation: six ChIP-seq datasets (~30 million reads each) processed in roughly 7 h 23 min on comparable hardware
- MultiQC report shows no unaddressed quality/contamination flags from FastQC/BBDuk filtering
- Config schema check: single-ended rows have 5 columns and pair-ended rows have 6, with no mixing in one file

## Limitations

- CIPHER only runs entire workflows — individual stages such as peak calling alone cannot be selected
- Single-ended and pair-ended data cannot be mixed in the same config file
- Running without a container requires all pipeline software to be installed locally; on a laptop/desktop, oversubscribing parallel processes (-qs) can exhaust memory and abort the run

## Evidence

- [abstract] narrow and broad peak calling, peak annotation, and motif identification for ChIP-seq: "narrow and broad peak calling, peak annotation, and motif identification for ChIP-seq"
- [readme] CIPHER is a data processing workflow platform for next generation sequencing data including ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq and GRO-seq.: "CIPHER is a data processing workflow platform for next generation sequencing data including ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq and GRO-seq."
- [readme] nextflow run cipher.nf -with-singularity <cipher.img> --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH>: [evidence span withheld — anchor status: external_doc]
- [results] CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [discussion] CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools: "CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools to complete certain tasks"
