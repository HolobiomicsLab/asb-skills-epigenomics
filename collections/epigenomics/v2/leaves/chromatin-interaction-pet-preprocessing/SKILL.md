---
name: chromatin-interaction-pet-preprocessing
description: Use when raw paired-end ChIA-PET FASTQ reads must be scanned for bridge
  linker sequences and partitioned by linker status prior to read mapping and loop
  calling.
when_to_use_negative:
- Input data are HiChIP or PLAC-seq reads only if their junction sequence between
  the two tags differs from the configured bridge linker — the linker scan must be
  configured for the library's junction sequence
- Reads are already linker-filtered or already aligned (e.g. BAM alignments with MAPQ
  ≥ 30) — re-running linker filtering would be redundant
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline shell that orchestrates the linker-filtering step as its
    first stage via 0.chia_pipe_shell.sh
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: ChIA-PET Utilities (CPU)
  role: Provides the linker_filter executable that scans reads for the bridge linker
    sequence and categorizes read pairs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:c4685f1adbbf18b99dacbe456265a4be0d5eda1b602dffd6e1ce2d153582ea11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - ChIA-PET Utilities (CPU)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chromatin-interaction-pet-preprocessing

## Summary

Scans paired-end ChIA-PET reads for the bridge linker sequence between the two genomic tags and partitions read pairs into linker-status categories before mapping, using ChIA-PIPE's CPU linker_filter module. This first pipeline step determines which PETs are eligible for downstream interaction-loop detection.

## When to use

Use when raw paired-end ChIA-PET FASTQ reads must be scanned for bridge linker sequences and partitioned by linker status prior to read mapping and loop calling.

## When NOT to use

- Input data are HiChIP or PLAC-seq reads only if their junction sequence between the two tags differs from the configured bridge linker — the linker scan must be configured for the library's junction sequence
- Reads are already linker-filtered or already aligned (e.g. BAM alignments with MAPQ ≥ 30) — re-running linker filtering would be redundant

## Inputs

- paired-end ChIA-PET FASTQ reads (fastq.gz, e.g. LDK0004-ds_*.fastq.gz)
- ChIA-PIPE configuration file (e.g. chia_pipe-master/example_config_file.sh)

## Outputs

- partitioned read-pair FASTQ files by linker category (no linker / one usable tag / full PET)
- linker_filter_stats.txt with counts per category

## How to apply

Place the paired-end ChIA-PET FASTQ files (e.g., LDK0004-ds_*.fastq.gz) in a fastq/ working directory and configure the ChIA-PIPE example config. Launch the pipeline via chia_pipe-master/0.chia_pipe_shell.sh, which invokes the CPU linker_filter module to scan every read for the bridge linker sequence between the two genomic tags. Each read pair is classified into one of three categories: (i) no linker sequence detected, (ii) a linker sequence and one usable genomic tag, or (iii) a linker sequence and paired-end tags (PETs) with a genuine bridge linker between both genomic tags. Partitioned read-pair files and per-category counts are written to linker_filter_stats.txt. Only the full-PET category (iii) read pairs are carried forward to read mapping and loop detection.

## Related tools

- **ChIA-PIPE** (Automated pipeline shell that orchestrates the linker-filtering step as its first stage via 0.chia_pipe_shell.sh) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **ChIA-PET Utilities (CPU)** (Provides the linker_filter executable that scans reads for the bridge linker sequence and categorizes read pairs) — https://github.com/cheehongsg/CPU

## Examples

```
mkdir -p fastq; cp LDK0004-ds_*.fastq.gz fastq/; bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- linker_filter_stats.txt reports counts for all three categories that together partition the total read pairs
- Only category (iii) full-PET read pairs (linker detected, both ends with genomic tags) are flagged as the set carried forward to loop detection
- Category (i) reads show no linker sequence; category (ii) reads contain a linker plus exactly one usable genomic tag
- The pipeline completes without error when launched from the working directory containing fastq/ and the example config

## Limitations

- Correct bridge linker sequence must be configured per library type; the pipeline is designed to handle differing junction sequences between ChIA-PET, HiChIP, and PLAC-seq but requires proper configuration
- Downstream loop detection uses only PETs with linker detected and both ends having genomic tags — categories (i) and (ii) are discarded from interaction analysis

## Evidence
