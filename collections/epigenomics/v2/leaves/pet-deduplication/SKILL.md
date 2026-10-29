---
name: pet-deduplication
description: Use when linker-filtered ChIA-PET FASTQ tags must be mapped to a reference
  genome and deduplicated so that downstream peak calling and loop calling receive
  only uniquely aligned, nonredundant PETs.
when_to_use_negative:
- Input is raw, unfiltered FASTQ reads that have not undergone ChIA-PIPE linker filtering
  — run the linker-filtering category step first.
- The task is calling peaks, loops, CCDs, or E-P annotations — this skill stops at
  analysis-ready PET BAMs and does not perform those downstream steps.
- The data are not PET-style chromatin interaction reads (e.g., ordinary ChIP-seq
  fragments) — the hybrid BWA-ALN/BWA-MEM and CPU dedup workflow is specific to paired-end
  tag junction reads.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Runs the memaln tag-alignment module using the hybrid BWA-ALN/BWA-MEM strategy
    and orchestrates the dedup step within the automated pipeline
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the "dedup" module that removes redundant PETs and retains unique
    MAPQ ≥ 30 alignments
  repo: https://github.com/cheehongsg/CPU
- name: BWA
  role: Aligns linker-flanked tag sequences to the reference genome via a hybrid of
    BWA-ALN and BWA-MEM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:cf9e0140704c6618060d23b6104dcb6eb5c7553b2419d971b0b2747dc25131a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BWA
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pet-deduplication

## Summary

Aligns linker-trimmed ChIA-PET tags to a reference genome with a hybrid BWA-ALN/BWA-MEM strategy and retains only unique (MAPQ ≥ 30), nonredundant PETs via the CPU "dedup" module, producing analysis-ready PET BAM alignments for downstream peak and loop calling.

## When to use

Use when linker-filtered ChIA-PET FASTQ tags must be mapped to a reference genome and deduplicated so that downstream peak calling and loop calling receive only uniquely aligned, nonredundant PETs.

## When NOT to use

- Input is raw, unfiltered FASTQ reads that have not undergone ChIA-PIPE linker filtering — run the linker-filtering category step first.
- The task is calling peaks, loops, CCDs, or E-P annotations — this skill stops at analysis-ready PET BAMs and does not perform those downstream steps.
- The data are not PET-style chromatin interaction reads (e.g., ordinary ChIP-seq fragments) — the hybrid BWA-ALN/BWA-MEM and CPU dedup workflow is specific to paired-end tag junction reads.

## Inputs

- Linker-filtered/trimmed ChIA-PET FASTQ files (e.g., LDK0004-ds_*.fastq.gz placed in the fastq directory)
- Reference genome for the target cells
- ChIA-PIPE configuration file (example_config_file.sh)

## Outputs

- BAM file of analysis-ready PETs containing only uniquely aligned (MAPQ ≥ 30), nonredundant PETs, ready for peak and loop calling

## How to apply

After ChIA-PIPE linker filtering, align read pairs from each category separately to the target reference genome using the hybrid BWA strategy, mapping the flanking sequences with a combination of BWA-ALN and BWA-MEM. Retain only unique alignments by applying the MAPQ ≥ 30 threshold. Remove redundant PETs by invoking the CPU "dedup" module (ChIA-PET Utilities). Run the memaln tag-alignment module on the linker-trimmed tags via 0.chia_pipe_shell.sh with example_config_file.sh (or the PBS qsub route on HPC), using the LDK0004-ds test FASTQs placed in the fastq directory. The rationale is that uniquely mapped, nonredundant PETs are the required substrate for downstream peak and loop detection; multimappers and PCR duplicates would inflate spurious interactions.

## Related tools

- **ChIA-PIPE** (Runs the memaln tag-alignment module using the hybrid BWA-ALN/BWA-MEM strategy and orchestrates the dedup step within the automated pipeline) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the "dedup" module that removes redundant PETs and retains unique MAPQ ≥ 30 alignments) — https://github.com/cheehongsg/CPU
- **BWA** (Aligns linker-flanked tag sequences to the reference genome via a hybrid of BWA-ALN and BWA-MEM)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- A valid BAM file of analysis-ready PETs is emitted by the memaln/dedup run.
- All retained alignments have mapping quality MAPQ ≥ 30.
- The BAM contains no redundant (duplicate) PETs — duplicates were removed by the CPU "dedup" module.
- PETs from each linker-filter category were aligned separately and only uniquely mapped reads were retained.
- The output BAM is usable as direct input for downstream ChIA-PIPE peak calling and loop calling.

## Limitations

- The reference genome version used for alignment is not specified in the discussion section of the article.
- Multimapping reads and PCR duplicates are discarded, so repetitive regions and over-amplified PETs are excluded from analysis-ready alignments.
- The test data (LDK0004-ds FASTQ files) must be obtained from the Dropbox deposit; pipeline availability depends on that external source and on local installation of ChIA-PIPE dependencies via local_install_chia_pipe_dependencies.sh.

## Evidence

- [discussion] Unique alignments and nonredundant PETs retained via CPU dedup: [evidence span withheld — anchor status: not_found]
