---
name: chia-pet-linker-filtering
description: Use when raw ChIA-PET, HiChIP, or PLAC-seq FASTQ read pairs (e.g., 2
  × 21 bp or 2 × 150 bp) must have their bridge linker sequences detected before mapping
  and downstream peak/loop calling.
when_to_use_negative:
- Input is already linker-filtered or aligned PET data (e.g., deduplicated BAM with
  MAPQ ≥ 30) — linker filtering would be redundant
- Chromatin-mapping assay with no bridge linker between tags (e.g., standard Hi-C),
  where linker categorization does not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline that executes linker filtering as the first step
    of ChIA-PET processing, prior to read mapping, peak calling, and loop calling
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Collection of modularized executables integrated by ChIA-PIPE that provides
    the efficient specialized programs for processing ChIA-PET data from raw reads
    to interactions
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:b36ed051e256a7cddf3ce0c8b96c254765304d97652922ef51352e6656aa64ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chia-pet-linker-filtering

## Summary

Identify and categorize bridge-linker sequences within raw ChIA-PET read pairs so that only PETs carrying a genuine linker between two genomic tags proceed to alignment, peak calling, and loop calling within ChIA-PIPE.

## When to use

Use when raw ChIA-PET, HiChIP, or PLAC-seq FASTQ read pairs (e.g., 2 × 21 bp or 2 × 150 bp) must have their bridge linker sequences detected before mapping and downstream peak/loop calling.

## When NOT to use

- Input is already linker-filtered or aligned PET data (e.g., deduplicated BAM with MAPQ ≥ 30) — linker filtering would be redundant
- Chromatin-mapping assay with no bridge linker between tags (e.g., standard Hi-C), where linker categorization does not apply

## Inputs

- Raw paired-end FASTQ read-pair files (e.g., LDK0004-ds_*.fastq.gz) placed in a fastq/ subdirectory
- ChIA-PIPE configuration file (e.g., example_config_file.sh) specifying library parameters
- Reference genome for the target cells used by the downstream BWA-ALN/BWA-MEM hybrid alignment

## Outputs

- Linker-filtered PETs categorized for genuine bridge linker sequence between the two genomic tags
- Linker-detected PETs with both genomic tags, suitable for read mapping and loop detection
- Automated quality control assessment for the dataset

## How to apply

Set up a working directory, clone ChIA-PIPE from GitHub, and install its dependencies with local_install_chia_pipe_dependencies.sh into a local dependency directory. Place raw read-pair files (e.g., LDK0004-ds_*.fastq.gz) into a fastq/ subdirectory as required by the configuration, then launch the pipeline via the shell script or the PBS HPC script; linker filtering is the first step executed before read mapping. Categorize read pairs for a genuine bridge linker sequence between the two genomic tags — a pipeline capable of handling the different junction sequences between the two tags is what allows the same code path to serve ChIA-PET, HiChIP, and PLAC-seq libraries. Only PET reads with a detected linker and genomic tags at both ends are carried forward to mapping (via BWA-ALN/BWA-MEM hybrid alignment) and, ultimately, to interaction-loop detection. Resource expectations: a ~300 million read-pair HFFc6 CTCF HiSeq library processed in ~10 wall-clock hours with 20 threads and 60 Gb RAM.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline that executes linker filtering as the first step of ChIA-PET processing, prior to read mapping, peak calling, and loop calling) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Collection of modularized executables integrated by ChIA-PIPE that provides the efficient specialized programs for processing ChIA-PET data from raw reads to interactions) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Only PET reads with linker detected and both ends having genomic tags are forwarded to the interaction-loop detection stage
- The pipeline run completes and generates the automated quality control assessment outputs for the test library
- Called peaks, interaction loops, and .hic/.cool visualization files are produced, confirming that filtering fed a valid downstream chain
- Downstream dedup retains only unique alignments with MAPQ ≥ 30 and nonredundant PETs, so linker-passing reads survive the CPU dedup module
- Wall-clock runtime and resources stay in the reported envelope (e.g., ~10 hours, 20 threads, 60 Gb RAM for ~300M read pairs)

## Limitations

- The reference genome version used for alignment of linker-flanking sequences is not specified in the article's methods description
- Massive libraries (hundreds of millions of PET reads) demand substantial compute; the pipeline requires robustness to handle >100M PET reads per library
- Different junction sequences between the two tags across ChIA-PET, HiChIP, and PLAC-seq must be accounted for by the categorization logic

## Evidence

- [readme] ChIA-PET Utilities is a collection of efficient specialized programs for processing ChIA-PET data from raw reads to interactions.: [evidence span withheld — anchor status: not_found]
