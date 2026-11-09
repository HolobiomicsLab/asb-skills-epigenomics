---
name: bam-file-generation
description: Use when linker-filtered ChIA-PET/HiChIP FASTQ tags must be mapped to
  a reference genome and reduced to unique, nonredundant PET alignments (MAPQ ≥ 30)
  in a BAM for downstream peak and loop calling.
when_to_use_negative:
- Input reads have not yet been linker-filtered/categorized — alignment of raw reads
  bypasses the genuine bridge-linker categorization step ChIA-PIPE depends on
- The goal is multi-mapping-sensitive analysis where reads with MAPQ < 30 are needed
  — this skill intentionally discards non-unique alignments
- The data type is not a ChIA-PET-family assay (e.g. standard ChIP-seq reads without
  PET structure), since deduplication and PET pairing logic assume paired end-tagged
  reads
edam_operation: http://edamontology.org/operation_0362
edam_topics: []
tools:
- name: ChIA-PIPE
  role: Orchestrates the memaln tag-alignment module using a hybrid BWA-ALN/BWA-MEM
    strategy on linker-trimmed tags
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the "dedup" module that retains unique (MAPQ ≥ 30) and nonredundant
    PETs in the output BAM
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:0a4375194ab7f40deb9beeb64b364310fbdbbae568b986beac15c5646f416acd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bam-file-generation

## Summary

Align linker-trimmed ChIA-PET tags to a reference genome using ChIA-PIPE's hybrid BWA-ALN/BWA-MEM strategy and deduplicate mapped PETs via the CPU "dedup" module to emit an analysis-ready BAM containing only uniquely aligned, nonredundant PET alignments.

## When to use

Use when linker-filtered ChIA-PET/HiChIP FASTQ tags must be mapped to a reference genome and reduced to unique, nonredundant PET alignments (MAPQ ≥ 30) in a BAM for downstream peak and loop calling.

## When NOT to use

- Input reads have not yet been linker-filtered/categorized — alignment of raw reads bypasses the genuine bridge-linker categorization step ChIA-PIPE depends on
- The goal is multi-mapping-sensitive analysis where reads with MAPQ < 30 are needed — this skill intentionally discards non-unique alignments
- The data type is not a ChIA-PET-family assay (e.g. standard ChIP-seq reads without PET structure), since deduplication and PET pairing logic assume paired end-tagged reads

## Inputs

- Linker-trimmed (linker-filtered) ChIA-PET FASTQ files, e.g. LDK0004-ds_*.fastq.gz placed in the fastq directory
- Reference genome index for BWA alignment of the target species
- ChIA-PIPE configuration file (example_config_file.sh)

## Outputs

- BAM file of analysis-ready PET alignments containing only uniquely mapped (MAPQ ≥ 30), nonredundant PETs, ready for peak and loop calling

## How to apply

After linker filtering, run ChIA-PIPE's memaln tag-alignment module on the trimmed tags, mapping the flanking genomic sequences with a hybrid of BWA-ALN and BWA-MEM against the reference genome of the target cells; each linker-filter category of read pairs is aligned separately. Retain only unique alignments with MAPQ ≥ 30. Then call the CPU "dedup" module to remove redundant (PCR-duplicate) PETs. The rationale is that only uniquely mapped and nonredundant tags should feed downstream chromatin interaction analysis. The pipeline is launched via 0.chia_pipe_shell.sh with example_config_file.sh (or the PBS qsub route on HPC), using test FASTQ files such as LDK0004-ds placed in the fastq directory.

## Related tools

- **ChIA-PIPE** (Orchestrates the memaln tag-alignment module using a hybrid BWA-ALN/BWA-MEM strategy on linker-trimmed tags) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the "dedup" module that retains unique (MAPQ ≥ 30) and nonredundant PETs in the output BAM) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- A valid BAM file of analysis-ready PETs is emitted by the pipeline
- All retained alignments have MAPQ ≥ 30 (uniquely mapped reads only)
- No redundant/duplicate PETs remain after the CPU "dedup" module runs
- The BAM is suitable for immediate downstream peak calling and loop calling by ChIA-PIPE
- Alignment was run on linker-filtered categories separately, using the hybrid BWA-ALN/BWA-MEM strategy, not a single aligner

## Limitations

- The reference genome version used for alignment is not specified in the discussion section of the article
- Reads with MAPQ < 30 and redundant PETs are permanently discarded, so the output BAM cannot be used for analyses requiring multi-mapping or duplicate reads
- Only uniquely mapped and nonredundant tags are retained before downstream chromatin interaction analysis, which may reduce complexity in repetitive regions

## Evidence

- [discussion] Hybrid BWA-ALN/BWA-MEM alignment of flanking sequences: "the flanking sequences are mapped to the reference genome of target cells using a hybrid of BWA-ALN (Burrows-Wheeler aligner) and BWA-MEM"
- [discussion] Unique, nonredundant PET retention via CPU dedup: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline launch command from README: "bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh"
- [abstract] Only uniquely mapped, nonredundant tags retained: [evidence span withheld — anchor status: not_found]
