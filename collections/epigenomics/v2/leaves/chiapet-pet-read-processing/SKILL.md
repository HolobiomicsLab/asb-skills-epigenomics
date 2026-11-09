---
name: chiapet-pet-read-processing
description: Use when raw ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must be converted
  into deduplicated PET alignments, peaks, and interaction loops with reproducible
  QC, as in the GM12878 CTCF MiSeq test data workflow.
when_to_use_negative:
- Input is already deduplicated PET alignments or a loop table — the linker filtering,
  BWA mapping, and CPU dedup stages would be redundant.
- Data are not chromatin-mapping reads (e.g., plain ChIP-seq FASTQ without PET linker
  structure) — no bridge linker or paired tags exist for PET processing.
- Only allele-specific phasing is needed and reads are already uniquely aligned and
  loop-called — use the haplotype-assignment/binomial-test workflow instead.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline driver for linker filtering, read mapping, peak calling,
    loop calling, and QC assessment
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities modular executables, including the 'dedup' module that
    retains unique MAPQ ≥ 30 nonredundant PETs
  repo: github.com/cheehongsg/CPU
- name: Juicebox
  role: Two-dimensional contact map viewing of ChIA-PIPE-generated .hic files
- name: HiGlass
  role: Two-dimensional contact map viewing of ChIA-PIPE-generated .cool files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:12b01700e5f3a53d4bda0b3b6efb28c2d229f0ff82509c5e9c10540cb62b4fe4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chiapet-pet-read-processing

## Summary

Process raw ChIA-PET paired-end tag (PET) reads into unique, peak-supported interaction loops using ChIA-PIPE/CPU: linker filtering, BWA mapping, MAPQ ≥ 30 dedup, peak calling, and loop calling (PET count ≥ 3 with peak support at both anchors).

## When to use

Use when raw ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must be converted into deduplicated PET alignments, peaks, and interaction loops with reproducible QC, as in the GM12878 CTCF MiSeq test data workflow.

## When NOT to use

- Input is already deduplicated PET alignments or a loop table — the linker filtering, BWA mapping, and CPU dedup stages would be redundant.
- Data are not chromatin-mapping reads (e.g., plain ChIP-seq FASTQ without PET linker structure) — no bridge linker or paired tags exist for PET processing.
- Only allele-specific phasing is needed and reads are already uniquely aligned and loop-called — use the haplotype-assignment/binomial-test workflow instead.

## Inputs

- Raw ChIA-PET FASTQ reads (e.g., LDK0004-ds_*.fastq.gz paired-end MiSeq data)
- Reference genome FASTQ/index for BWA alignment (GM12878/NA12878 hg19 context)
- Input-control sample for peak calling (recommended to reduce false-positive peaks)

## Outputs

- Unique, nonredundant PET alignments (MAPQ ≥ 30)
- Peak calls (self-ligation PET-derived binding peaks)
- Interaction loops with PET count ≥ 3 and peak support at both anchors
- .hic and .cool visualization files for Juicebox and HiGlass

## How to apply

Stage raw reads (e.g., LDK0004-ds_*.fastq.gz) in a fastq/ directory and run the ChIA-PIPE driver script (0.chia_pipe_shell.sh) with the example configuration. The pipeline categorizes read pairs for genuine bridge linker sequence between the two genomic tags, maps flanking genomic tags with a hybrid of BWA-ALN and BWA-MEM, and retains unique alignments (MAPQ ≥ 30) and nonredundant PETs via the CPU 'dedup' module. Peaks are called (with input-control support to reduce false positives), then loops are called with PET count ≥ 3 and peak support at both anchors. PETs spanning ≤ 8 kb are classified as self-ligation; only PET reads with detected linker and both genomic tags feed loop detection. Outputs include .hic/.cool files for Juicebox and HiGlass visualization.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline driver for linker filtering, read mapping, peak calling, loop calling, and QC assessment) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities modular executables, including the 'dedup' module that retains unique MAPQ ≥ 30 nonredundant PETs) — github.com/cheehongsg/CPU
- **Juicebox** (Two-dimensional contact map viewing of ChIA-PIPE-generated .hic files)
- **HiGlass** (Two-dimensional contact map viewing of ChIA-PIPE-generated .cool files)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Mapping/dedup filters applied correctly: retained alignments all have MAPQ ≥ 30 and no redundant PETs remain after CPU 'dedup'.
- Loop-calling invariants: every exported loop has PET count ≥ 3 and peak support at both anchors.
- Self-ligation/interligation classification consistent with the 8-kb genomic span cutoff.
- Pipeline completes end-to-end on the Dropbox test data (LDK0004-ds fastq files) without QC flags from ChIA-PIPE's automated quality assessment.
- Downstream allele-specific check: the phased CTCF loop table from the processed reads reproduces the reported 636 phased-interaction loops.

## Limitations

- Peak calling on very large datasets without an input-control sample results in an abundance of false-positive peaks; use input control and stringent defaults.
- Custom E-P/P-P classification script and ChIA-PIPE version are not stated in the article's discussion; verify tool versions independently.
- Reference genome version for alignment and accession IDs for the allele-specific CTCF/RNAPII data are not specified in the article section — confirm genome build before running.
- Older pipelines' approaches motivated ChIA-PIPE's defaults; mixing parameters across pipelines may invalidate expected loop/peak counts.

## Evidence

- [discussion] MAPQ and dedup retention: [evidence span withheld — anchor status: not_found]
