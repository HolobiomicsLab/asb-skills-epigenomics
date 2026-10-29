---
name: mapq-threshold-filtering
description: Use when ChIA-PET (or HiChIP/PLAC-seq) read pairs have been linker-filtered
  and aligned with the hybrid BWA-ALN/BWA-MEM strategy, and analysis-ready unique,
  nonredundant PETs must be selected for peak/loop calling.
when_to_use_negative:
- Input reads have not yet been linker-filtered and categorized — run the linker filtering
  step first
- The goal is to inspect raw read quality or linker detection statistics rather than
  produce analysis-ready alignments
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Orchestrates hybrid BWA-ALN/BWA-MEM tag alignment (memaln module) and launches
    the dedup filtering workflow
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: Provides the 'dedup' module that removes redundant PETs after MAPQ >= 30 unique-alignment
    filtering
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:ecb799cc8142e079ffeb1d10a897d64732af6f6ba4da5a99df9adb541addf180
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mapq-threshold-filtering

## Summary

A ChIA-PIPE filtering step that retains only uniquely mapped PET alignments with MAPQ >= 30 and removes redundant PETs via the CPU 'dedup' module, yielding analysis-ready PET alignments for downstream peak and loop calling.

## When to use

Use when ChIA-PET (or HiChIP/PLAC-seq) read pairs have been linker-filtered and aligned with the hybrid BWA-ALN/BWA-MEM strategy, and analysis-ready unique, nonredundant PETs must be selected for peak/loop calling.

## When NOT to use

- Input reads have not yet been linker-filtered and categorized — run the linker filtering step first
- The goal is to inspect raw read quality or linker detection statistics rather than produce analysis-ready alignments

## Inputs

- Linker-trimmed FASTQ files (e.g., LDK0004-ds test FASTQs in the fastq directory)
- Reference genome index for BWA-ALN/BWA-MEM
- ChIA-PIPE configuration file (example_config_file.sh)

## Outputs

- BAM file of analysis-ready PETs containing only uniquely aligned (MAPQ >= 30), nonredundant PETs

## How to apply

After linker filtering, align read pairs from each category separately to the reference genome using ChIA-PIPE's hybrid BWA strategy (BWA-ALN and BWA-MEM, invoked through the memaln tag-alignment module). Retain only unique alignments with mapping quality (MAPQ) >= 30, as configured in example_config_file.sh and run via 0.chia_pipe_shell.sh (or the PBS qsub route on HPC). Then remove redundant PETs by calling the CPU 'dedup' module. The rationale is to eliminate ambiguously mapped and PCR-duplicated PETs so that only uniquely mapped, nonredundant tags feed downstream chromatin interaction (peak and loop) analysis.

## Related tools

- **ChIA-PIPE** (Orchestrates hybrid BWA-ALN/BWA-MEM tag alignment (memaln module) and launches the dedup filtering workflow) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (Provides the 'dedup' module that removes redundant PETs after MAPQ >= 30 unique-alignment filtering) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Output BAM file is emitted and is a valid BAM
- All retained alignments have MAPQ >= 30
- BAM contains no redundant (duplicate) PETs after CPU dedup
- Retained PETs are uniquely mapped and suitable for downstream peak and loop calling

## Limitations

- Reference genome version used for alignment is not specified in the source section
- Only uniquely mapped reads are retained, so multi-mapping PETs near repetitive regions are discarded

## Evidence

- [discussion] Retain only unique alignments with MAPQ ≥ 30 and nonredundant PETs (CPU dedup): [evidence span withheld — anchor status: not_found]
