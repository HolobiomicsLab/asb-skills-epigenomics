---
name: linker-filtering-qc-count-reporting
description: Use when starting from raw paired-end ChIA-PET FASTQ reads (e.g., LDK0004-ds_*.fastq.gz)
  and you must detect the bridge linker and categorize read pairs before mapping.
when_to_use_negative:
- Input reads are already linker-filtered or already aligned (e.g., BAM files); this
  skill operates on raw paired-end FASTQ reads
- Library design has no bridge linker junction between genomic tags, so the three-way
  linker categorization does not apply
edam_operation: http://edamontology.org/operation_0232
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose first step runs linker filtering on the raw ChIA-PET
    reads via the CPU linker_filter module
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Collection of modularized executables that performs the linker detection and
    read-pair categorization
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:4d096e49e5e65228146a004e2fac08340d7f386dd3e6e4470461a1eaf19dffaf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_0232
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# linker-filtering-qc-count-reporting

## Summary

First ChIA-PIPE processing step that scans each paired-end ChIA-PET read for the bridge linker sequence between the two genomic tags and partitions read pairs into three categories, reporting per-category counts for quality control.

## When to use

Use when starting from raw paired-end ChIA-PET FASTQ reads (e.g., LDK0004-ds_*.fastq.gz) and you must detect the bridge linker and categorize read pairs before mapping.

## When NOT to use

- Input reads are already linker-filtered or already aligned (e.g., BAM files); this skill operates on raw paired-end FASTQ reads
- Library design has no bridge linker junction between genomic tags, so the three-way linker categorization does not apply

## Inputs

- Paired-end ChIA-PET FASTQ files in fastq/ (e.g., LDK0004-ds_*.fastq.gz)
- ChIA-PIPE configuration file (e.g., chia_pipe-master/example_config_file.sh)

## Outputs

- Partitioned read-pair FASTQ files for categories (i) no linker, (ii) linker with one usable tag, (iii) full PET with bridge linker
- linker_filter_stats.txt with counts for each category

## How to apply

Place the paired-end ChIA-PET test reads (LDK0004-ds_*.fastq.gz) in a fastq/ working directory and launch ChIA-PIPE's CPU linker_filter module via chia_pipe-master/0.chia_pipe_shell.sh with the example config. The module scans every read for the bridge linker sequence located between the two genomic tags. Each read pair is classified into one of three categories: (i) no linker sequence, (ii) a linker sequence and one usable genomic tag, or (iii) a linker sequence and full PETs (paired-end tags) with a genuine bridge linker between both genomic tags. Partitioned read-pair files are written and the per-category counts are tallied into linker_filter_stats.txt. Only category (iii) read pairs are carried forward to read mapping and loop detection, making this count split the basis for downstream QC.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose first step runs linker filtering on the raw ChIA-PET reads via the CPU linker_filter module) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Collection of modularized executables that performs the linker detection and read-pair categorization) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- linker_filter_stats.txt reports counts for all three categories and the counts partition the total read pairs
- Only full-PET (category iii) read pairs are flagged as the set carried forward to read mapping and loop detection
- Category (i) plus category (ii) plus category (iii) counts sum exactly to the input read-pair count
- Downstream mapping receives the category (iii) files, not the raw FASTQ inputs

## Limitations

- Correct classification depends on supplying the right junction/linker sequence in the config; ChIA-PIPE, HiChIP, and PLAC-seq data have different junction sequences between the two tags
- Read pairs lacking a detected linker (category i) or with only one usable tag (category ii) are excluded from loop detection, so poor ligation or sequencing quality directly reduces the usable PET fraction

## Evidence

- [readme] CPU provides the modularized executables for this step: [evidence span withheld — anchor status: external_doc]
