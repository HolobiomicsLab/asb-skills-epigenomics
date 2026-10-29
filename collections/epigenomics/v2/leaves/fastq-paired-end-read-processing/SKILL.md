---
name: fastq-paired-end-read-processing
description: Use when raw paired-end ChIA-PET FASTQ files (e.g. bridge-linker protocol
  data such as GSE72816/GSM970213) must be stripped of the 5' bridge linker before
  read alignment.
when_to_use_negative:
- Input is half-linker ChIA-PET data — the bridge-linker split-linkerA/linkerB settings
  apply to the bridge-linker protocol; half-linker data requires ChIA-PET2's other
  mode
- Input is already trimmed reads or downstream files (SAM/BAM/bedpe) — use the later
  pipeline steps (read alignment, removeDup, buildBedpe) instead
- Linker sequence is unknown or a different protocol's linker is used — supplying
  the CTCF bridge-linker sequences verbatim would mis-trim other datasets
edam_operation: http://edamontology.org/operation_3259
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ChIA-PET2
  role: trimLinker subcommand trims bridge-linker sequences from pair-end fastq(.gz)
    files and writes the trim.stat QC summary
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: downstream short-read aligner for the trimmed paired-end reads
  repo: https://github.com/lh3/bwa
- name: samtools
  role: SAM/BAM handling after alignment of trimmed reads
  repo: https://github.com/samtools/samtools
- name: bedtools
  role: bed/bedpe file manipulation following PET construction
  repo: https://github.com/arq5x/bedtools2
- name: MACS2
  role: peak calling on tagAlign built from trimmed-read PETs
  repo: https://github.com/macs3-project/MACS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:a7b050b4f76da40589c778c6797f6633b3e005a406e5dee37ab205e52fcf3af9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - MACS2
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_3259
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fastq-paired-end-read-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trims adapter/linker sequences from paired-end ChIA-PET FASTQ reads with ChIA-PET2's trimLinker in bridge-linker mode, so that downstream alignment, PET filtering, duplicate removal, peak calling and loop calling operate on clean tag sequences.

## When to use

Use when raw paired-end ChIA-PET FASTQ files (e.g. bridge-linker protocol data such as GSE72816/GSM970213) must be stripped of the 5' bridge linker before read alignment.

## When NOT to use

- Input is half-linker ChIA-PET data — the bridge-linker split-linkerA/linkerB settings apply to the bridge-linker protocol; half-linker data requires ChIA-PET2's other mode
- Input is already trimmed reads or downstream files (SAM/BAM/bedpe) — use the later pipeline steps (read alignment, removeDup, buildBedpe) instead
- Linker sequence is unknown or a different protocol's linker is used — supplying the CTCF bridge-linker sequences verbatim would mis-trim other datasets

## Inputs

- Paired-end ChIA-PET raw read FASTQ files (or fastq.gz) from a bridge-linker protocol dataset (e.g. GM12878 CTCF ChIA-PET, GEO GSE72816 / GSM970213)
- Bridge linker sequence (ACGCGATATCTTATCTGACT), split into linkerA=ACGCGATATCTTATC and linkerB=AGTCAGATAAGATAT

## Outputs

- Trimmed paired-end FASTQ reads with linker sequences removed
- output.trim.stat summary file reporting trimmed/rescued PET counts and valid PET ratio

## How to apply

Run ChIA-PET2's trimLinker subcommand on the pair-end fastq(.gz) files in bridge-linker mode (-m 1), supplying the first 15 bases of each 5' end of the bridge linker: linkerA=ACGCGATATCTTATC and linkerB=AGTCAGATAAGATAT. Start strict with 0 allowed mismatches (-e 0), then repeat with -e 1 to rescue PETs with one linker mismatch, keeping default keep-empty (-k 0) and minimum trimmed read length 15 (-l 15). Parse the output.trim.stat summary of each run to compare rescued PET counts and the valid PET ratio. The rationale is that one-mismatch tolerance rescued >16 million PETs on the GM12878 CTCF dataset, improving the valid PET ratio by 12.85%, whereas strict matching discards these reads before alignment (with bwa), duplicate removal, peak calling (MACS2) and loop calling.

## Related tools

- **ChIA-PET2** (trimLinker subcommand trims bridge-linker sequences from pair-end fastq(.gz) files and writes the trim.stat QC summary) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (downstream short-read aligner for the trimmed paired-end reads) — https://github.com/lh3/bwa
- **samtools** (SAM/BAM handling after alignment of trimmed reads) — https://github.com/samtools/samtools
- **bedtools** (bed/bedpe file manipulation following PET construction) — https://github.com/arq5x/bedtools2
- **MACS2** (peak calling on tagAlign built from trimmed-read PETs) — https://github.com/macs3-project/MACS

## Examples

```
ChIA-PET2 trimLinker -m 1 -e 1 -k 0 -l 15 -a ACGCGATATCTTATC -b AGTCAGATAAGATAT -f GSM970213_R1.fq.gz -r GSM970213_R2.fq.gz
```

## Evaluation signals

- output.trim.stat from the -e 1 run reports approximately >16 million rescued PETs versus the -e 0 run on the GM12878 CTCF dataset
- Valid PET ratio improves by roughly 12.85% when one linker mismatch is allowed relative to 0 mismatches
- Trimmed reads meet the minimum length threshold (-l 15) so they remain alignable
- PET counts pass the QC gate before downstream steps: read alignment, PET filtering, PCR duplicate removal, peak calling and loop calling complete on the trimmed output

## Limitations

- No versions are stated for ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2, g++), so exact trimming behavior may vary between builds
- Allowing more than one mismatch in the linker search was not benchmarked in this study; the reported rescue applies to -e 1
- Loop calling depends on ChIP-Seq-like peak calling sensitivity, which constrains how much trimming tolerance ultimately improves final loops
- The pipeline requires a linux-like system with OpenMP support

## Evidence
