---
name: chipseq-read-mapping
description: Use when raw ChIPseq FASTQ reads must be aligned to a reference genome
  inside an automated pipeline, especially when standard and spike-in (exogenous epigenome)
  samples must be aligned with identical parameters for ChIP-rx normalization.
when_to_use_negative:
- Reads are already aligned (.bam available) — proceed directly to the 'filter', 'cleanfiles'
  or 'GRanges' modules instead of re-mapping
- The data are RNAseq rather than ChIPseq — although NEAT supports RNAseq, this skill
  is scoped to ChIPseq read mapping feeding peak calling and ChIP-rx normalization
- A non-NEAT, fully scripted workflow with custom aligner parameters is required —
  NEAT exposes alignment only through the centralized Targets.txt project file
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NEAT
  role: Pipeline framework providing the double-clickable 'map' module and the centralized
    Targets.txt that sets alignment parameters
  repo: https://github.com/pschorderet/NEAT
- name: bwa
  role: Primary read aligner invoked by the 'map' module
- name: bowtie
  role: Alternative read aligner selectable in the 'map' module
- name: SAMtools
  role: Handles aligned reads downstream (duplicate-read handling and fragment-size
    filtering in the 'filter' module; .bam/.bai reorganization via 'cleanfiles')
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:01a7698521b91b4d8e845fdc2295d39da4707216ec7a178b64a3a41dc9ab0e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - R
  - SAMtools
  - bowtie
  - bwa
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# chipseq-read-mapping

## Summary

Align ChIPseq reads within the NEAT pipeline using the 'map' module, which maps reads with bwa or bowtie and feeds aligned reads into the filter, cleanfiles and peakcalling stages. Mapping parameters are centralized in the Targets.txt project file so every sample is aligned identically.

## When to use

Use when raw ChIPseq FASTQ reads must be aligned to a reference genome inside an automated pipeline, especially when standard and spike-in (exogenous epigenome) samples must be aligned with identical parameters for ChIP-rx normalization.

## When NOT to use

- Reads are already aligned (.bam available) — proceed directly to the 'filter', 'cleanfiles' or 'GRanges' modules instead of re-mapping
- The data are RNAseq rather than ChIPseq — although NEAT supports RNAseq, this skill is scoped to ChIPseq read mapping feeding peak calling and ChIP-rx normalization
- A non-NEAT, fully scripted workflow with custom aligner parameters is required — NEAT exposes alignment only through the centralized Targets.txt project file

## Inputs

- Unzipped and renamed FASTQ read files (produced by the 'unzip' module)
- Targets.txt project file with sample names, replicates, conditions, antibodies and alignment parameters
- Reference genome index for bwa or bowtie
- NEAT truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells (for validation)

## Outputs

- Aligned reads (BAM files)
- BAI index files (reorganized by the 'cleanfiles' module)
- Alignments ready for the 'filter' and 'peakcalling' modules

## How to apply

Define sample names, replicates, conditions, antibodies and alignment parameters in the single centralized Targets.txt file that a NEAT project is built around, then run the 'map' module (one of NEAT's double-clickable executables: unzip, rename, QC, chiprx, map, filter, peakcalling). The map module maps reads using either bwa or bowtie; choose one aligner and apply the same parameters to both the standard H3K4me3 ChIPseq sample and the spike-in epigenome, since ChIP-RX requires consistency between the two alignments. Downstream, the 'filter' module uses AdvancedSettings.txt (duplicate-read handling, minimum and maximum fragment size) and the 'cleanfiles' module reorganizes the resulting .bam and .bai files before downstream analysis. Validate by running the module end-to-end on the truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells shipped with the NEAT repository.

## Related tools

- **NEAT** (Pipeline framework providing the double-clickable 'map' module and the centralized Targets.txt that sets alignment parameters) — https://github.com/pschorderet/NEAT
- **bwa** (Primary read aligner invoked by the 'map' module)
- **bowtie** (Alternative read aligner selectable in the 'map' module)
- **SAMtools** (Handles aligned reads downstream (duplicate-read handling and fragment-size filtering in the 'filter' module; .bam/.bai reorganization via 'cleanfiles'))

## Evaluation signals

- The module runs end-to-end on the truncated H3K4me3 ChIPseq test dataset without errors
- Both the standard and the spike-in (exogenous epigenome) samples are aligned with the same bwa/bowtie mapping parameters, as ChIP-RX requires consistency between the two alignments
- Alignment parameters used match the entries in the centralized Targets.txt file rather than ad-hoc settings
- Downstream modules accept the output: 'filter' processes the alignments with AdvancedSettings.txt parameters and 'cleanfiles' successfully reorganizes the resulting .bam and .bai files

## Limitations

- NEAT's documented operating system is Mac OSx only; no Linux/Windows support is stated
- NEAT is still under development (README: 'If you encounter any bug, please contact us') and no changelog is provided
- No concrete quantitative results are claimed for the pipeline; tangible results are only outputs (Fig. 3), so mapping quality must be judged from downstream module behavior

## Evidence

- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The 'map' module maps reads using either bwa [18] or bowtie [19].: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [readme] NEAT is a next generation analysis toolkit that supports the analysis of large data including metagene analysis (ChIPseq) and differential gene expression analysis (RNAseq).: "NEAT is a next generation analysis toolkit that supports the analysis of large data including metagene analysis (ChIPseq)"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
