---
name: paired-end-read-pair-categorization
description: Use when raw paired-end ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must
  be scanned for the bridge linker junction and partitioned by linker/tag content
  before read mapping.
when_to_use_negative:
- Input reads have already been linker-filtered and split into usable tags — re-running
  would re-categorize already-trimmed PETs.
- Data are not junction-ligated chromatin contact reads (e.g., plain RNA-seq or ChIP-seq
  FASTQ with no bridge linker between tags).
- The goal is downstream filtering such as MAPQ ≥ 30 deduplication or self-ligation
  vs interligation classification — those are separate CPU modules (dedup, 8-kb span
  cutoff), not linker filtering.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline that orchestrates the linker-filtering step and partitions
    read pairs before mapping, peak calling, and loop calling
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: ChIA-PET Utilities (CPU)
  role: Collection of modularized executables (e.g., linker_filter) that performs
    the bridge-linker scan and three-way categorization of read pairs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:aeec086df02bc98e02bb238718f63ef779679653d357ebcfa16da558b97d235d
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
yes_rate: 0.875
n_claims: 8
---

# paired-end-read-pair-categorization

## Summary

ChIA-PIPE's first processing step (CPU linker_filter): scan each paired-end ChIA-PET read for the bridge linker sequence between the two genomic tags and partition read pairs into three categories — no linker, one usable tag, or full PETs — so that only category (iii) proceeds to loop detection.

## When to use

Use when raw paired-end ChIA-PET (or HiChIP/PLAC-seq) FASTQ reads must be scanned for the bridge linker junction and partitioned by linker/tag content before read mapping.

## When NOT to use

- Input reads have already been linker-filtered and split into usable tags — re-running would re-categorize already-trimmed PETs.
- Data are not junction-ligated chromatin contact reads (e.g., plain RNA-seq or ChIP-seq FASTQ with no bridge linker between tags).
- The goal is downstream filtering such as MAPQ ≥ 30 deduplication or self-ligation vs interligation classification — those are separate CPU modules (dedup, 8-kb span cutoff), not linker filtering.

## Inputs

- paired-end ChIA-PET FASTQ read pairs (e.g., LDK0004-ds_*.fastq.gz) in a fastq/ working directory
- ChIA-PIPE configuration file (e.g., chia_pipe-master/example_config_file.sh) specifying the bridge linker sequence and library settings

## Outputs

- partitioned read-pair FASTQ files for the three linker/tag categories
- linker_filter_stats.txt summary tallying counts per category
- full-PET (category iii) read set flagged for downstream mapping and loop detection

## How to apply

Place the paired-end ChIA-PET test reads (e.g., LDK0004-ds_*.fastq.gz) in a fastq/ working directory alongside a ChIA-PIPE configuration file, then launch the linker-filtering step (ChIA-PIPE's CPU linker_filter module) via chia_pipe-master/0.chia_pipe_shell.sh with the example config. The module scans every read for the bridge linker sequence between the two genomic tags and classifies each read pair into one of three categories: (i) no linker sequence, (ii) a linker sequence and one usable genomic tag, or (iii) a linker sequence and paired-end tags (PETs). Partitioned read-pair files are written and per-category counts are tallied in linker_filter_stats.txt. Only the full-PET (category iii) read pairs — those with a genuine bridge linker between both genomic tags — are flagged as the set carried forward to mapping and loop detection.

## Related tools

- **ChIA-PIPE** (Automated pipeline that orchestrates the linker-filtering step and partitions read pairs before mapping, peak calling, and loop calling) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **ChIA-PET Utilities (CPU)** (Collection of modularized executables (e.g., linker_filter) that performs the bridge-linker scan and three-way categorization of read pairs) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- linker_filter_stats.txt reports all three categories and their counts sum to the total number of input read pairs (a partition invariant).
- Only the category (iii) full-PET read pairs (linker detected with both ends having genomic tags) are flagged and carried forward to loop detection.
- Category (i) contains no bridge-linker sequence; category (ii) contains a linker plus exactly one usable genomic tag.
- The step completes when launched through chia_pipe-master/0.chia_pipe_shell.sh with the example config on the LDK0004-ds test FASTQs without errors.
- Downstream inputs to read mapping derive solely from the retained full-PET set, not from categories (i) or (ii).

## Limitations

- Only PET reads in category (iii) are used for detection of interaction loops; reads with no linker or a single usable tag are discarded from loop calling.
- The junction sequence scanned depends on the assay (ChIA-PET vs HiChIP/PLAC-seq), so the linker definition must match the library type in the configuration.
- Reference genome version and ChIA-PIPE version/Docker identifier are not stated in the described section, complicating exact reproducibility.

## Evidence
