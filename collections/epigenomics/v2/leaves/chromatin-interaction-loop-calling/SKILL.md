---
name: chromatin-interaction-loop-calling
description: Use when processing ChIA-PET, HiChIP, or PLAC-seq read pairs (short 2×21
  bp or long 2×150 bp) and you need to call chromatin interaction loops with peak-supported
  anchors from uniquely aligned, nonredundant PETs.
when_to_use_negative:
- Input is raw ChIA-PET FASTQ not yet processed for linker filtering and read mapping
  — loop calling requires uniquely aligned, deduplicated PETs (MAPQ ≥ 30).
- The data type is a plain chromatin contact map or Hi-C matrix rather than ligated
  PET reads with bridge linkers; ChIA-PIPE's linker-based loop detection is not appropriate.
- The dataset is too small to yield loops with PET count ≥ 3 and peak support, in
  which case loop calls would be unreliable.
edam_operation: http://edamontology.org/operation_3221
edam_topics: []
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline orchestrating linker filtering, read mapping, peak
    calling, and loop calling on ChIA-PET data
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Collection of modularized executables (e.g. 'dedup') that filter unique alignments
    and nonredundant PETs feeding loop detection
  repo: https://github.com/cheehongsg/CPU
- name: Juicebox
  role: Web-based 2D contact map viewer consuming .hic/.cool visualization files generated
    after loop calling
- name: HiGlass
  role: Web-based 2D contact map viewer consuming .hic/.cool visualization files generated
    after loop calling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:07e8c09ef911b945e4beec868813cdcc2c6d2ae408902d14dc65c98e750a76df
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_3221
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromatin-interaction-loop-calling

## Summary

ChIA-PIPE's loop-calling stage identifies chromatin interaction loops (anchor-to-anchor PETs) from ChIA-PET/HiChIP data after linker filtering, read mapping, peak calling, and duplicate removal, then classifies PETs as self-ligation versus interligation and annotates loops as enhancer-promoter or promoter-promoter.

## When to use

Use when processing ChIA-PET, HiChIP, or PLAC-seq read pairs (short 2×21 bp or long 2×150 bp) and you need to call chromatin interaction loops with peak-supported anchors from uniquely aligned, nonredundant PETs.

## When NOT to use

- Input is raw ChIA-PET FASTQ not yet processed for linker filtering and read mapping — loop calling requires uniquely aligned, deduplicated PETs (MAPQ ≥ 30).
- The data type is a plain chromatin contact map or Hi-C matrix rather than ligated PET reads with bridge linkers; ChIA-PIPE's linker-based loop detection is not appropriate.
- The dataset is too small to yield loops with PET count ≥ 3 and peak support, in which case loop calls would be unreliable.

## Inputs

- ChIA-PET / HiChIP / PLAC-seq paired-end FASTQ read-pair files (e.g. LDK0004-ds_*.fastq.gz) that have passed linker filtering and read mapping
- Uniquely aligned PETs with MAPQ ≥ 30 after CPU dedup
- Called peak calls from the peak-calling stage for anchor support

## Outputs

- Chromatin interaction loops (PET count ≥ 3, peak-supported anchors) with unique loop IDs
- Self-ligation vs. interligation PET classification (8-kb genomic span cutoff)
- Chromatin contact domains (CCDs) of at least 25 kb with continuous loop coverage
- Enhancer-promoter vs. promoter-promoter loop annotations from a custom Python script
- Input files for 2D contact map viewing (.hic and .cool) with Juicebox and HiGlass

## How to apply

Run ChIA-PIPE (github.com/TheJacksonLaboratory/ChIA-PIPE), which integrates ChIA-PET Utilities (CPU) modules, after the upstream linker filtering, read mapping, and peak calling stages have completed. Retain only unique alignments with MAPQ ≥ 30 and nonredundant PETs via the CPU 'dedup' module, then classify PETs by genomic span: PETs spanning ≤ 8 kb are classified as self-ligation, while longer-spanning PETs are used for interaction loop detection. Restrict loop detection to PET reads with a genuine bridge linker detected and both ends having genomic tags. Require each called loop to have a PET count ≥ 3 and peak support; loops can then feed chromatin contact domain (CCD) calling, which retains only the top one-third of loops by PET count (67th percentile cutoff) and defines CCDs as ≥ 25 kb regions of continuous loop coverage.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline orchestrating linker filtering, read mapping, peak calling, and loop calling on ChIA-PET data) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Collection of modularized executables (e.g. 'dedup') that filter unique alignments and nonredundant PETs feeding loop detection) — https://github.com/cheehongsg/CPU
- **Juicebox** (Web-based 2D contact map viewer consuming .hic/.cool visualization files generated after loop calling)
- **HiGlass** (Web-based 2D contact map viewer consuming .hic/.cool visualization files generated after loop calling)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Only PET reads in category (iii) — with linker detected and both ends having genomic tags — are used for detection of interaction loops; verify PET category filtering was applied.
- Every retained PET has MAPQ ≥ 30 and is nonredundant (CPU 'dedup' module output).
- All called loops have PET count ≥ 3 and peak-supported anchors, and each loop carries a unique ID.
- Pipeline completes and produces loop calls, peaks, QC assessment outputs, and .hic/.cool visualization files for the test library (e.g. the LDK0004-ds dataset).
- CCD output regions are at least 25 kb with continuous loop coverage and are built from the top one-third of loops by PET count (67th percentile cutoff).

## Limitations

- Performing peak calling on very large datasets without an input-control sample, as older pipelines do, produces an abundance of false-positive peaks that propagate into loop calls; ChIA-PIPE uses input-control peak calling with stringent defaults.
- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location in the article.
- Reference genome version used for alignment (BWA-ALN/BWA-MEM hybrid) is not specified in the article's methods sections quoted here.

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) ≥ 30] and nonredundant PETs are retained by calling the CPU 'dedup' module.: [evidence span withheld — anchor status: not_found]
