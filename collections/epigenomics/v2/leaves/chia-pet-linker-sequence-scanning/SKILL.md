---
name: chia-pet-linker-sequence-scanning
description: Use when raw paired-end ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must
  be classified by bridge-linker detection before genome mapping, so only genuine
  linker-flanked PETs carry forward to interaction analysis.
when_to_use_negative:
- Reads are already linker-filtered PETs (e.g., deduplicated alignments with MAPQ
  >= 30) and only downstream mapping/peak/loop calling is needed
- Data are not linker-junction-based chromatin conformation assays (the article notes
  ChIA-PIPE targets ChIA-PET, HiChIP, and PLAC-seq junction sequences between two
  tags)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose shell script orchestrates the linker-filtering step
    as the first processing stage
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: ChIA-PET Utilities (CPU)
  role: Collection of modularized executables providing the CPU linker_filter module
    that scans reads for the bridge linker sequence between the two genomic tags
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:9a37ce805b2625cfb79e876c074d58997d1f67ef9ada553a57be5955f5db0e19
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# chia-pet-linker-sequence-scanning

## Summary

Scans each paired-end ChIA-PET read for the bridge linker sequence and partitions read pairs into three categories: (i) no linker, (ii) linker with one usable genomic tag, or (iii) linker with full paired-end tags (PETs). This is ChIA-PIPE's first processing step and determines which read pairs proceed to loop detection.

## When to use

Use when raw paired-end ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must be classified by bridge-linker detection before genome mapping, so only genuine linker-flanked PETs carry forward to interaction analysis.

## When NOT to use

- Reads are already linker-filtered PETs (e.g., deduplicated alignments with MAPQ >= 30) and only downstream mapping/peak/loop calling is needed
- Data are not linker-junction-based chromatin conformation assays (the article notes ChIA-PIPE targets ChIA-PET, HiChIP, and PLAC-seq junction sequences between two tags)

## Inputs

- Paired-end ChIA-PET FASTQ reads (e.g., LDK0004-ds_*.fastq.gz test data)
- ChIA-PIPE configuration file (e.g., chia_pipe-master/example_config_file.sh)

## Outputs

- Partitioned read-pair FASTQ files for the three linker categories
- linker_filter_stats.txt tallying counts of each category
- Set of full-PET (category iii) read pairs carried forward to mapping and loop detection

## How to apply

Place the paired-end ChIA-PET test reads (LDK0004-ds_*.fastq.gz) in a fastq/ working directory alongside an example ChIA-PIPE configuration file. Launch the linker-filtering step via ChIA-PIPE's CPU linker_filter module through chia_pipe-master/0.chia_pipe_shell.sh with the config; the module scans every read for the bridge linker sequence between the two genomic tags. Classify each read pair into one of three categories: (i) no linker detected, (ii) linker detected with one usable genomic tag, or (iii) linker detected with both genomic tags (full PET). Write the partitioned read-pair files and per-category counts to linker_filter_stats.txt. Only category (iii) full-PET read pairs are flagged as the set carried forward to loop detection; downstream, those PETs undergo dedup retaining unique alignments with MAPQ >= 30.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose shell script orchestrates the linker-filtering step as the first processing stage) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **ChIA-PET Utilities (CPU)** (Collection of modularized executables providing the CPU linker_filter module that scans reads for the bridge linker sequence between the two genomic tags) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- linker_filter_stats.txt reports all three categories with counts that sum to the total number of input read pairs
- Only category (iii) full-PET read pairs (linker detected with both genomic tags) are flagged as the set carried forward to loop detection
- Each read pair is assigned to exactly one of the three categories (no linker, one usable tag, full PET), i.e., the partition is exhaustive and non-overlapping
- Partitioned FASTQ files are produced per category and can be consumed by subsequent mapping steps of the pipeline

## Limitations

- Category (ii) reads yield only one usable genomic tag and cannot be used for loop detection; only category (iii) PETs proceed
- Detection depends on the expected bridge linker sequence; different assays (ChIA-PET vs HiChIP/PLAC-seq) have different junction sequences between the two tags, so the config must match the assay

## Evidence

- [discussion] ChIA-PIPE's first step partitions read pairs into three categories: those with (i) no linker sequence, (ii) a linker sequence and one usable genomic tag, or (iii) a linker sequence and PETs.: [evidence span withheld — anchor status: not_found]
