---
name: wiggle-signal-file-generation
description: Use when a ChIPseq pipeline step must produce a .wig genome-browser signal
  track (e.g. for IGV), especially when normalized genome-wide signal from standard
  plus spike-in (exogenous epigenome) alignments is needed.
when_to_use_negative:
- The goal is peak calling or GRanges object generation rather than a continuous browser
  signal track — use the 'peakcalling' or 'GRanges' modules instead
- The data are RNAseq rather than ChIPseq; wig/signal tracks here target ChIPseq genome-wide
  enrichment
- The platform is not Mac OSx, since the article's discussion section states the operating
  system is limited to Mac OSx with no Linux/Windows support stated
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NEAT
  role: Provides the 'Wig' pipeline module that writes genome-wide ChIPseq signal
    to .wig files, invoked via double-clickable executables or cluster submission
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Filters the standard and spike-in .bam alignments (view/sort/rmdup/index)
    with parameters from AdvancedSettings.txt before signal generation
- name: bwa
  role: Maps reads of standard and spike-in epigenomes with the same mapping parameters,
    as ChIP-RX requires consistency between the two alignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:df84efa2ff4a1da5f609aab75e02c5aa85f04cb7f1b87bd09f0ee758e61b92db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - R
  - SAMtools
  - bwa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# wiggle-signal-file-generation

## Summary

Generates genome-browser-compatible wiggle (.wig) signal files from ChIPseq alignments within the NEAT pipeline, optionally applying ChIP-rx spike-in normalization so genome-wide signal is quantitatively comparable across epigenomes.

## When to use

Use when a ChIPseq pipeline step must produce a .wig genome-browser signal track (e.g. for IGV), especially when normalized genome-wide signal from standard plus spike-in (exogenous epigenome) alignments is needed.

## When NOT to use

- The goal is peak calling or GRanges object generation rather than a continuous browser signal track — use the 'peakcalling' or 'GRanges' modules instead
- The data are RNAseq rather than ChIPseq; wig/signal tracks here target ChIPseq genome-wide enrichment
- The platform is not Mac OSx, since the article's discussion section states the operating system is limited to Mac OSx with no Linux/Windows support stated

## Inputs

- Filtered .bam alignments of standard ChIPseq samples (e.g. H3K4me3 from mouse embryonic stem cells)
- Filtered .bam alignments of spike-in (exogenous epigenome) ChIPseq samples
- NEAT Targets.txt project file with sample names, replicates, conditions, antibodies, and spike-in entries
- AdvancedSettings.txt with filtering parameters (duplicate-read handling, minimum and maximum fragment size)

## Outputs

- Genome-browser-compatible wiggle (.wig) signal file with normalized genome-wide ChIPseq signal

## How to apply

Run the NEAT 'Wig' module as one of the double-clickable pipeline modules alongside unzip, rename, QC, chiprx, map, filter, and peakcalling. If quantitative ChIP-rx normalization is required, first align standard and spike-in samples identically with bwa or bowtie (the 'map' module), then filter both alignments with samtools using the duplicate-read handling and minimum/maximum fragment-size parameters set in AdvancedSettings.txt. Apply the previously published ChIP-rx algorithm, scaling genome-wide ChIPseq signal by the defined quantity of the exogenous spike-in epigenome, and write the resulting genome-wide signal to a .wig file. Project-wide sample names, replicates, conditions, antibodies, and parameters come from NEAT's centralized project file (Targets.txt), including spike-in control entries or dash ('-') markers where no spike-in is used.

## Related tools

- **NEAT** (Provides the 'Wig' pipeline module that writes genome-wide ChIPseq signal to .wig files, invoked via double-clickable executables or cluster submission) — https://github.com/pschorderet/NEAT
- **SAMtools** (Filters the standard and spike-in .bam alignments (view/sort/rmdup/index) with parameters from AdvancedSettings.txt before signal generation)
- **bwa** (Maps reads of standard and spike-in epigenomes with the same mapping parameters, as ChIP-RX requires consistency between the two alignments)

## Evaluation signals

- The module script runs end-to-end on the truncated H3K4me3 ChIPseq test dataset shipped with the NEAT repository without errors
- The output .wig file is non-empty and browser-compatible (e.g. loadable in IGV)
- The signal scaling in the .wig file derives from the spike-in (exogenous epigenome) control, not from the standard alignment alone
- Standard and spike-in alignments used identical mapping (bwa/bowtie) and filtering (samtools, AdvancedSettings.txt) parameters
- The output appears in the NEAT project folder alongside archived pipeline outputs, per the automated and reproducible project structure

## Limitations

- NEAT is still under development (per README 'Devel mode'); bugs should be reported to the authors
- ChIP-rx normalization requires a properly defined exogenous spike-in epigenome with a defined quantity; entries without spike-in are marked with a dash ('-') and cannot be quantitatively scaled
- The article's results section offers no concrete quantitative results claims for the outputs beyond Fig. 3 figures

## Evidence

- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] ChIPrx [17] is a cutting edge normalization method for ChIPseq that performs genome-wide quantitative comparisons: "ChIPrx [17] is a cutting edge normalization method for ChIPseq that performs genome-wide quantitative comparisons"
- [abstract] Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive: "Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive"
- [readme] NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline ... (viii) Wig: "NEAT has been developed as four main modules:"
