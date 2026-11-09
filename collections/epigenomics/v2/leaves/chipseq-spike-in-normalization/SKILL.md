---
name: chipseq-spike-in-normalization
description: Use when ChIPseq samples must be quantitatively compared genome-wide
  and an exogenous spike-in epigenome (ChIP-rx) control was sequenced, or when reconstructing/running
  the NEAT 'chiprx' module.
when_to_use_negative:
- No exogenous spike-in epigenome was sequenced — standard ChIPseq samples without
  ChIP-rx controls cannot be scaled by spike-in quantity
- The goal is RNAseq analysis (e.g., differentially expressed gene analysis) rather
  than ChIPseq signal normalization
- Only peak-level (binary) comparisons are needed rather than genome-wide quantitative
  ChIPseq signal comparisons
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: NEAT
  role: Framework providing the automated 'chiprx' module for ChIP-rx normalization
    within the ChIPseq pipeline
  repo: https://github.com/pschorderet/NEAT
- name: bwa
  role: Maps reads for both standard and spike-in epigenome samples with the same
    mapping parameters
- name: SAMtools
  role: Filters aligned reads (view/sort/rmdup/index) identically for standard and
    spike-in alignments
- name: R
  role: Supports NEAT modules and downstream analysis via the R systemPipeR package
    used by the QC module
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:d3c03a53f8d21d916e8a0670c54eb312a1847b40d5975e785c73930b76368d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - R
  - SAMtools
  - bwa
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chipseq-spike-in-normalization

## Summary

Applies ChIP-rx, a genome-wide ChIPseq normalization method that scales signal by a defined quantity of an exogenous spike-in epigenome, enabling quantitative comparison of ChIPseq signal across samples. In NEAT this is packaged as the automated 'chiprx' module within its ChIPseq pipeline.

## When to use

Use when ChIPseq samples must be quantitatively compared genome-wide and an exogenous spike-in epigenome (ChIP-rx) control was sequenced, or when reconstructing/running the NEAT 'chiprx' module.

## When NOT to use

- No exogenous spike-in epigenome was sequenced — standard ChIPseq samples without ChIP-rx controls cannot be scaled by spike-in quantity
- The goal is RNAseq analysis (e.g., differentially expressed gene analysis) rather than ChIPseq signal normalization
- Only peak-level (binary) comparisons are needed rather than genome-wide quantitative ChIPseq signal comparisons

## Inputs

- Truncated H3K4me3 ChIPseq FASTQ test dataset from mouse embryonic stem cells (shipped with the NEAT repository)
- Targets.txt template with spike-in (exogenous epigenome) control entries
- AdvancedSettings.txt with filtering parameters (duplicate-read handling, minimum and maximum fragment size)
- Aligned BAM files for both standard and spike-in epigenome samples

## Outputs

- Normalized genome-wide ChIPseq signal in wiggle (.wig) format compatible with genome browsers such as IGV

## How to apply

Load the NEAT project's Targets.txt template shipped with the truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells, entering spike-in (exogenous epigenome) control entries and dash ('-') parameters where no spike-in is used. Map both the standard and spike-in epigenome samples with identical parameters using bwa or bowtie, since ChIP-RX requires consistency between the two alignments. Filter both alignments identically with samtools view/sort/rmdup/index using parameters from AdvancedSettings.txt (duplicate-read handling, minimum and maximum fragment size). Implement the ChIP-rx algorithm as previously published, scaling genome-wide ChIPseq signal by the defined quantity of the spike-in epigenome, and write the normalized signal to a wiggle (.wig) file for genome browsers such as IGV.

## Related tools

- **NEAT** (Framework providing the automated 'chiprx' module for ChIP-rx normalization within the ChIPseq pipeline) — https://github.com/pschorderet/NEAT
- **bwa** (Maps reads for both standard and spike-in epigenome samples with the same mapping parameters)
- **SAMtools** (Filters aligned reads (view/sort/rmdup/index) identically for standard and spike-in alignments)
- **R** (Supports NEAT modules and downstream analysis via the R systemPipeR package used by the QC module) — https://github.com/tgirke/systemPipeR

## Evaluation signals

- The chiprx module script runs end-to-end on the truncated H3K4me3 test dataset without errors
- The output .wig file is non-empty and openable in IGV
- The scaling of the normalized genome-wide signal demonstrably derives from the spike-in control quantity in Targets.txt
- Mapping and filtering parameters are identical between standard and spike-in epigenome alignments (same bwa/bowtie parameters, same AdvancedSettings.txt filter settings)
- The module is invocable alongside the other NEAT pipeline modules (unzip, rename, QC, map, filter, peakcalling) via double-clickable executables

## Limitations

- ChIP-RX requires consistency between standard and spike-in alignments — the same mapping parameters (bwa or bowtie) must be used for both
- NEAT is still under development and run as double-clickable executables on Mac OSx, with no Linux/Windows support stated and no changelog found in the repository
- The paper presents a pipeline whose tangible results are outputs (Fig. 3); no concrete quantitative performance results are stated for the chiprx module

## Evidence

- [intro] ChIP-rx listed as automated pipeline module runnable via double-clickable icons: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] ChIP-rx is a cutting edge normalization method for ChIPseq enabling genome-wide quantitative comparisons: "ChIPrx [17] is a cutting edge normalization method for ChIPseq that performs genome-wide quantitative comparisons"
- [methods] Filter module parameters come from AdvancedSettings.txt including duplicate reads and fragment size bounds: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [readme] README lists ChIP-rx normalization as pipeline module (iii): "(iii)	ChIP-rx normalization"
- [methods] Map module uses bwa or bowtie: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
