---
name: chia-pet-tag-alignment
description: Use when linker-filtered ChIA-PET FASTQ tags must be mapped to a reference
  genome and deduplicated into analysis-ready PETs before peak and loop calling.
when_to_use_negative:
- Input FASTQ files have not yet undergone linker filtering — run ChIA-PIPE's linker
  filtering step first to categorize read pairs before alignment.
- The data type is not ChIA-PET-like (e.g., standard WGS or RNA-seq reads); the hybrid
  BWA-ALN/BWA-MEM strategy here is specific to linker-flanked ChIA-PET tag structure.
- The goal is already-deduplicated PET processing such as peak calling or loop calling
  — this skill only produces the analysis-ready BAM consumed by those later modules.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline whose memaln module performs hybrid BWA-ALN/BWA-MEM
    tag alignment of linker-trimmed tags
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the 'dedup' module invoked to retain unique alignments (MAPQ >= 30)
    and remove nonredundant PETs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:4f323fd3790067b4fa636b0c69f5d3fba0843bd376d6d5149ee741e89eaa7d81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chia-pet-tag-alignment

## Summary

Align linker-trimmed ChIA-PET tags to a reference genome using ChIA-PIPE's hybrid BWA-ALN/BWA-MEM strategy, then retain only unique alignments (MAPQ >= 30) and remove redundant PETs with the CPU 'dedup' module to produce analysis-ready PET alignments.

## When to use

Use when linker-filtered ChIA-PET FASTQ tags must be mapped to a reference genome and deduplicated into analysis-ready PETs before peak and loop calling.

## When NOT to use

- Input FASTQ files have not yet undergone linker filtering — run ChIA-PIPE's linker filtering step first to categorize read pairs before alignment.
- The data type is not ChIA-PET-like (e.g., standard WGS or RNA-seq reads); the hybrid BWA-ALN/BWA-MEM strategy here is specific to linker-flanked ChIA-PET tag structure.
- The goal is already-deduplicated PET processing such as peak calling or loop calling — this skill only produces the analysis-ready BAM consumed by those later modules.

## Inputs

- linker-trimmed FASTQ files of ChIA-PET read pairs (e.g., LDK0004-ds_*.fastq.gz in the fastq directory)
- reference genome index for BWA
- ChIA-PIPE configuration file (example_config_file.sh) specifying alignment parameters

## Outputs

- BAM file of analysis-ready PETs containing only uniquely aligned, nonredundant PET alignments ready for downstream peak and loop calling

## How to apply

After linker filtering categorizes read pairs, align the trimmed tags in each category separately to the reference genome using ChIA-PIPE's memaln tag-alignment module, which maps the flanking genomic tag sequences with a hybrid of BWA-ALN and BWA-MEM. Retain only unique alignments with MAPQ (mapping quality) >= 30, discarding multi-mapping reads. Then remove redundant PETs by calling the CPU 'dedup' module to eliminate PCR duplicates. The rationale is that only uniquely mapped and nonredundant tags should feed downstream chromatin interaction analysis (peak and loop calling), since redundant or ambiguously mapped PETs inflate interaction counts and create false positives. Launch the module via 0.chia_pipe_shell.sh with example_config_file.sh (or the PBS qsub route on HPC) using test FASTQ files such as the LDK0004-ds set placed in the fastq directory.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline whose memaln module performs hybrid BWA-ALN/BWA-MEM tag alignment of linker-trimmed tags) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the 'dedup' module invoked to retain unique alignments (MAPQ >= 30) and remove nonredundant PETs) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- A valid BAM file of analysis-ready PETs is emitted after the dedup module completes.
- All retained alignments in the BAM have MAPQ >= 30, confirming only unique alignments were kept.
- The BAM contains only nonredundant (deduplicated) PETs — re-running dedup yields no additional removals.
- PET counts decrease monotonically from linker-filtered input through unique-alignment filtering to deduplicated output, as expected.
- The output BAM is accepted by downstream ChIA-PIPE peak-calling and loop-calling modules without schema or index errors.

## Limitations

- The reference genome version used for alignment is not specified in the article's discussion section.
- Only uniquely mapped reads (MAPQ >= 30) are retained, so multi-mapping tags in repetitive regions are discarded and contribute no interactions.
- Short-read (2 x 21 bp) ChIA-PET tags rely on BWA-ALN while longer tags use BWA-MEM; misconfiguration of the hybrid strategy may misalign very short tags.

## Evidence

- [discussion] MAPQ 30 and dedup: [evidence span withheld — anchor status: not_found]
- [other] unique and nonredundant tags retained: [evidence span withheld — anchor status: not_found]
