---
name: chia-pet-read-mapping
description: Use when processing raw ChIA-PET (or related HiChIP/PLAC-seq) FASTQ read
  pairs that have passed linker filtering and need to be aligned to a reference genome
  before peak and loop calling.
when_to_use_negative:
- Input reads have not yet been linker-filtered — ChIA-PIPE requires the linker-filtering
  stage to run before mapping
- The data are not ChIA-PET-type chromatin-mapping reads (e.g. plain RNA-seq or WGS
  FASTQs) — this pipeline's linker and PET logic does not apply
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0102
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline orchestrating linker filtering, read mapping, peak
    calling, and loop calling; invoked via 0.chia_pipe_shell.sh or 0.chia_pipe_hpc.pbs
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the 'dedup' module that retains unique MAPQ >= 30, nonredundant PETs
    after mapping
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:a7504060d243954b6aad1298ebd1c336f23a8a52fb7d9182e1c011f774bebe6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# chia-pet-read-mapping

## Summary

Map linker-filtered ChIA-PET read pairs to a reference genome and retain high-confidence unique alignments as the mapping stage of the ChIA-PIPE pipeline, which feeds downstream PET classification, peak calling, and loop calling.

## When to use

Use when processing raw ChIA-PET (or related HiChIP/PLAC-seq) FASTQ read pairs that have passed linker filtering and need to be aligned to a reference genome before peak and loop calling.

## When NOT to use

- Input reads have not yet been linker-filtered — ChIA-PIPE requires the linker-filtering stage to run before mapping
- The data are not ChIA-PET-type chromatin-mapping reads (e.g. plain RNA-seq or WGS FASTQs) — this pipeline's linker and PET logic does not apply

## Inputs

- Linker-filtered ChIA-PET FASTQ read pairs (e.g. LDK0004-ds_*.fastq.gz from the ChIA-PIPE test dataset)
- Reference genome for the target cell type
- ChIA-PIPE configuration file (e.g. example_config_file.sh)

## Outputs

- Uniquely aligned PETs with MAPQ >= 30
- Nonredundant (deduplicated) PETs from the CPU 'dedup' module
- Self-ligation vs interligation classified PETs (8-kb genomic span cutoff)
- Downstream inputs for peak calling and loop calling; .hic and .cool visualization files

## How to apply

As part of ChIA-PIPE, run read mapping after linker filtering by launching the pipeline with the shell script (0.chia_pipe_shell.sh) or the HPC PBS script (0.chia_pipe_hpc.pbs), both driven by a configuration file such as example_config_file.sh. The flanking sequences resulting from linker filtering are mapped to the reference genome of the target cells using a hybrid of BWA-ALN and BWA-MEM, which lets one pipeline handle both short-read (2 x 21 bp) and long-read (2 x 150 bp) ChIA-PET data. After alignment, retain only unique alignments with MAPQ >= 30 and remove redundant PETs by calling the CPU 'dedup' module, since very large libraries (hundreds of millions of PET reads) otherwise yield abundant artifacts. Correctly mapped, deduplicated PETs are then classified by genomic span (self-ligation PETs <= 8 kb vs interligation PETs) and only PETs with detected linkers and genomic tags on both ends proceed to loop detection.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline orchestrating linker filtering, read mapping, peak calling, and loop calling; invoked via 0.chia_pipe_shell.sh or 0.chia_pipe_hpc.pbs) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the 'dedup' module that retains unique MAPQ >= 30, nonredundant PETs after mapping) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Mapping completes within the full ChIA-PIPE run producing called peaks, interaction loops, QC assessment outputs, and .hic/.cool files
- Only unique alignments with MAPQ >= 30 are retained and redundant PETs are removed by CPU 'dedup'
- PETs are partitioned by the 8-kb genomic span cutoff into self-ligation and interligation classes
- Loop detection uses only PETs with detected linkers and genomic tags on both ends
- Automated quality control assessment reports acceptable metrics for the mapped dataset

## Limitations

- Peak calling on very large datasets without an input-control sample results in an abundance of false-positive peaks, so the mapping output alone is not sufficient for reliable peaks
- The reference genome version used for alignment is not specified in the article's discussion section
- Massive libraries (~300 million read pairs) require substantial compute — about 10 wall-clock hours with 20 threads and 60 Gb RAM

## Evidence

- [discussion] Retain only unique alignments with MAPQ >= 30 and nonredundant PETs (CPU dedup): [evidence span withheld — anchor status: not_found]
