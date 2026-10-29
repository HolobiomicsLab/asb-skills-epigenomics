---
name: modular-ngs-pipeline-scripting
description: Use when a lab must turn raw NGS data (FASTQ from ChIPseq or RNAseq of
  mouse embryonic stem cells or similar) into reproducible QC, alignment, filtering,
  peak-calling and signal-track outputs with minimal command-line expertise.
when_to_use_negative:
- The analysis goal requires fine-grained interactive parameter exploration or custom
  scripting at each step — NEAT targets non-expert users running predefined modules,
  unlike a flexible web platform such as Galaxy.
- 'The host environment is not Mac OSx-compatible: the article''s discussion lists
  only ''Operating system: Mac OSx'' and states no Linux/Windows support.'
- Only the mapping step is needed in isolation — the skill's value is vertical integration
  managing interdependencies between unzip, QC, chiprx, map, filter and peakcalling,
  not single-step execution.
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: NEAT
  role: Framework providing the double-clickable automated pipeline modules (unzip,
    rename, QC, chiprx, map, filter, peakcalling, wig, GRanges, cleanfiles) and the
    centralized project/settings files
  repo: https://github.com/pschorderet/NEAT
- name: R
  role: Hosts the systemPipeR package used by the NEAT 'QC' module and downstream
    exploratory data analysis (metagenomic, differentially expressed gene analysis)
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: 'Aligner invoked by the NEAT ''map'' module (alternative: bowtie) to map reads
    of the standard and spike-in epigenome samples with identical parameters'
- name: SAMtools
  role: Read-manipulation backend supporting the alignment/filtering modules (e.g.
    duplicate-read handling in the 'filter' module via AdvancedSettings.txt parameters)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:ca1987c426acbec945c83650a25aa4731a203203585a3440dd6db1472e380d81
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# modular-ngs-pipeline-scripting

## Summary

Structure a ChIPseq/RNAseq analysis as a set of discrete, individually launchable NEAT modules (unzip, rename, QC, chiprx, map, filter, peakcalling, wig, GRanges) driven by one centralized project file, so wet-lab scientists can run fully automated NGS pipelines on a cluster via double-clickable executables without programming skills.

## When to use

Use when a lab must turn raw NGS data (FASTQ from ChIPseq or RNAseq of mouse embryonic stem cells or similar) into reproducible QC, alignment, filtering, peak-calling and signal-track outputs with minimal command-line expertise.

## When NOT to use

- The analysis goal requires fine-grained interactive parameter exploration or custom scripting at each step — NEAT targets non-expert users running predefined modules, unlike a flexible web platform such as Galaxy.
- The host environment is not Mac OSx-compatible: the article's discussion lists only 'Operating system: Mac OSx' and states no Linux/Windows support.
- Only the mapping step is needed in isolation — the skill's value is vertical integration managing interdependencies between unzip, QC, chiprx, map, filter and peakcalling, not single-step execution.

## Inputs

- Truncated H3K4me3 ChIPseq FASTQ test dataset from mouse embryonic stem cells (shipped with the NEAT repository)
- RNAseq FASTQ test dataset from mouse embryonic stem cells
- Targets.txt project file (sample names, replicates, conditions, antibodies, spike-in control entries or '-' if none)
- AdvancedSettings.txt with alignment-, filtering- and peak-calling parameters and cluster settings
- Reference genome index for bwa or bowtie alignment

## Outputs

- Sorted, duplicate-managed, indexed .bam/.bai alignment files (reorganized by the cleanfiles module)
- Peak calls from the selected peak-calling algorithm
- Genome-wide signal tracks as wiggle (.wig) and bigwig files, including ChIP-rx-normalized signal
- Small GRanges objects for downstream exploratory analysis (metagenomic plots, differential gene expression, pdf graphs, xls sheets, venn diagrams)

## How to apply

Create a NEAT project around a single centralized file holding sample names, replicates, conditions, antibodies, alignment-, filtering- and peak-calling parameters and cluster settings; spike-in (exogenous epigenome) control entries are recorded there, with dash ('-') parameters where no spike-in is used. Build the pipeline by chaining the automated modules in order: unzip/rename the FASTQ files, run QC via the R systemPipeR package, optionally apply ChIP-rx normalization (chiprx, the published genome-wide spike-in scaling method), map reads with bwa or bowtie, filter with parameters from AdvancedSettings.txt (duplicate-read handling, minimum and maximum fragment size), call peaks, then produce wig/bigwig files and compact GRanges objects for downstream analysis. Each module is a double-clickable executable that can run locally or be submitted to the institution's cluster (qsub or bsub), where storage space, maximum job submissions, wall time and cluster-specific parameters are customizable. The 'cleanfiles' module reorganizes .bam/.bai files before downstream steps, keeping outputs small and shareable. Follow the tutorials in the repository's 'Vignette' folder with the shipped truncated H3K4me3 ChIPseq or RNAseq test datasets before running real data.

## Related tools

- **NEAT** (Framework providing the double-clickable automated pipeline modules (unzip, rename, QC, chiprx, map, filter, peakcalling, wig, GRanges, cleanfiles) and the centralized project/settings files) — https://github.com/pschorderet/NEAT
- **R** (Hosts the systemPipeR package used by the NEAT 'QC' module and downstream exploratory data analysis (metagenomic, differentially expressed gene analysis)) — https://github.com/tgirke/systemPipeR
- **bwa** (Aligner invoked by the NEAT 'map' module (alternative: bowtie) to map reads of the standard and spike-in epigenome samples with identical parameters)
- **SAMtools** (Read-manipulation backend supporting the alignment/filtering modules (e.g. duplicate-read handling in the 'filter' module via AdvancedSettings.txt parameters))

## Examples

```
git clone https://github.com/pschorderet/NEAT; follow the tutorials in the 'Vignette' folder with the shipped truncated ChIPseq test dataset, launching each module (unzip, QC, chiprx, map, filter, peakcalling) via its double-clickable executable or qsub/bsub on the cluster
```

## Evaluation signals

- The pipeline runs end-to-end on the repository's truncated H3K4me3 ChIPseq (or RNAseq) test dataset without errors, as prescribed in the 'Vignette' tutorials.
- Each module's expected output exists and is non-empty: renamed FASTQ folder, QC report, sorted/indexed .bam plus .bai files, peak calls, and a non-empty .wig signal file.
- When the chiprx module is used with spike-in controls, the normalized .wig scaling derives from the exogenous spike-in epigenome; with no spike-in, the Targets.txt entries carry '-' parameters.
- The entire run completes within the reported <24 hour bound and uses only parameters captured in the single centralized project file, confirming reproducibility.
- Downstream GRanges objects are significantly smaller than the .bam files yet sufficient for metagenomic and differential-expression analyses (pdf graphs, xls sheets).

## Limitations

- NEAT is still under development; the README asks users to report bugs to the developers.
- The discussion section lists only Mac OSx as the operating system, with no stated Linux or Windows support.
- No concrete quantitative results are presented beyond pipeline outputs (Fig. 3), no version number is stated, and the repository has no changelog, so reproducibility across versions is unverified.
- Each pipeline completes as batch modules on the institution's cluster with customizable job-submission limits and wall time; platforms like Galaxy are positioned as the comparison, not drop-in alternatives.

## Evidence

- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive: "Through the development of double-clickable executables, NEAT is efficient (completes within <24 hours), easy to implement and intuitive"
- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [readme] NEAT has been developed as four main modules:: "NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
