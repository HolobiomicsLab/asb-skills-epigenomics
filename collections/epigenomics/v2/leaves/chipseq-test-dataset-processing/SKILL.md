---
name: chipseq-test-dataset-processing
description: Use when a NEAT project must be exercised end-to-end on the bundled truncated
  H3K4me3 ChIPseq test fastq files (Targets.txt registered), or when validating that
  NEAT's double-clickable pipeline modules run without programming experience.
when_to_use_negative:
- Input is RNAseq test data needing differential gene expression analysis rather than
  the ChIPseq modules (chiprx, peakcalling)
- The dataset is full-scale (untruncated) and a strict <2 h-per-dataset runtime acceptance
  criterion is required — NEAT's <24 h claim was measured on truncated test data
- The environment is not Mac OSx or a qsub/bsub cluster — the discussion notes the
  operating system is limited to Mac OSx
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3678
tools:
- name: NEAT
  role: framework providing the double-clickable modules (unzip, QC, chiprx, map,
    filter, peakcalling, cleanfiles, granges) and the centralized Targets.txt project
    file
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package underlying the NEAT 'QC' module that computes the per-cycle
    quality and k-mer diversity plots
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: alignment backend of the 'map' module (alternative to bowtie)
- name: bowtie
  role: alignment backend of the 'map' module (alternative to bwa)
- name: samtools
  role: filter module operations (view/sort/rmdup/index) on mapped reads
- name: MACS
  role: peak calling algorithm selectable in the 'peakcalling' module (alternative
    to SPP)
- name: SPP
  role: peak calling algorithm selectable in the 'peakcalling' module (alternative
    to MACS)
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:ee65df19a2843856f428a1c36f65c18a59e73300d794c609f9da625ebb4bd7d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - MACS
  - NEAT
  - R
  - SPP
  - Tophat
  - bowtie
  - bwa
  - samtools
  - systemPipeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chipseq-test-dataset-processing

## Summary

Run NEAT's end-to-end automated ChIPseq modules (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) on the truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells bundled with the NEAT package, producing bam, GRanges and wig outputs plus human-readable QC reports.

## When to use

Use when a NEAT project must be exercised end-to-end on the bundled truncated H3K4me3 ChIPseq test fastq files (Targets.txt registered), or when validating that NEAT's double-clickable pipeline modules run without programming experience.

## When NOT to use

- Input is RNAseq test data needing differential gene expression analysis rather than the ChIPseq modules (chiprx, peakcalling)
- The dataset is full-scale (untruncated) and a strict <2 h-per-dataset runtime acceptance criterion is required — NEAT's <24 h claim was measured on truncated test data
- The environment is not Mac OSx or a qsub/bsub cluster — the discussion notes the operating system is limited to Mac OSx

## Inputs

- truncated H3K4me3 ChIPseq test fastq files (mouse embryonic stem cells) bundled with NEAT from github.com/pschorderet/NEAT
- Targets.txt file with OriFileName and FileName columns
- AdvancedSettings.txt with alignment, filtering and peak calling parameters

## Outputs

- fatsq folder with decompressed and renamed fastq files
- QC pdf report (per-cycle quality, base proportion, k-mer diversity, read length distribution, mean quality) matching Fig. 3a
- .bam/.bai files organized by the cleanfiles module
- GRanges objects (~40-60 Mb regime)
- .wig files (~4-6 Mb regime)
- neat_benchmark.csv summarizing runtime and file sizes per dataset

## How to apply

Obtain the truncated H3K4me3 ChIPseq test fastq files bundled with NEAT from pschorderet/NEAT and register them in the centralized Targets.txt file (OriFileName/FileName columns). Create the project with the 'New Project' application and specify the Steps_to_execute_pipe blocks as unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges. Launch the pipeline via the double-clickable executable or the command line: the unzip module decompresses, renames and stores fatsq files in the project's fatsq folder; the QC module (built on the R systemPipeR package) produces per-cycle quality box plots, base proportion, relative k-mer diversity, read length/occurrence distribution and mean quality distribution as pdf reports mirroring Fig. 3a; map aligns reads with bwa or bowtie; filter manages duplicate reads and fragment size limits via AdvancedSettings.txt; peakcalling calls peaks with MACS or SPP; and cleanfiles plus granges reorganize .bam/.bai files and build smaller GRanges objects and wiggle files. Validation is by existence and completeness of the QC report categories and by file-size regimes (wig ~4-6 Mb smallest, GRanges ~40-60 Mb, bam ~4-6 Gb largest for full-scale data), since the paper states no numeric QC threshold.

## Related tools

- **NEAT** (framework providing the double-clickable modules (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) and the centralized Targets.txt project file) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package underlying the NEAT 'QC' module that computes the per-cycle quality and k-mer diversity plots) — https://github.com/tgirke/systemPipeR
- **bwa** (alignment backend of the 'map' module (alternative to bowtie))
- **bowtie** (alignment backend of the 'map' module (alternative to bwa))
- **samtools** (filter module operations (view/sort/rmdup/index) on mapped reads)
- **MACS** (peak calling algorithm selectable in the 'peakcalling' module (alternative to SPP))
- **SPP** (peak calling algorithm selectable in the 'peakcalling' module (alternative to MACS))

## Evaluation signals

- QC report artifacts exist for every registered test sample and contain the specified plot categories (per-cycle quality, base proportion, relative k-mer diversity, read length/occurrence distribution, mean quality) mirroring Fig. 3a
- Output file classes (bam, GRanges, wig) are produced for the truncated dataset with wig files smallest and bam files largest, matching the size regimes reported in the paper (wig ~4-6 Mb; GRanges ~40-60 Mb; bam ~4-6 Gb for full-scale)
- The truncated analysis completes in less than two hours per dataset (paper's overall claim is completion within <24 hours)
- The fatsq folder is created inside the project directory and contains the decompressed, renamed fastq files
- Runtime and file-size measurements for both ChIPseq and RNAseq test datasets compile into a single summary table (neat_benchmark.csv)

## Limitations

- The paper reports no numeric QC threshold; acceptance is existence and completeness of QC output categories, not metric cutoffs
- Operating system limited to Mac OSx; no Linux/Windows support stated
- NEAT version number and repository changelog are not provided; the toolkit is still under development and bug reports must be sent to the authors
- Test datasets are truncated, so runtime and file-size results scale differently for full-scale data

## Evidence

- [methods] truncated test datasets: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [methods] unzip module: "The ‘unzip’ module will unzip, rename and store fatsq files in a newly created folder within the project folder."
- [methods] QC module: "The ‘QC’ module uses the R systemPipeR package"
- [methods] map module: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [intro] automated modules list: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] test dataset tutorial: "Novice users can follow through the entire analysis of a test dataset provided as part of the package. Please follow the tutorials found in the 'vignette' folder"
- [readme] run modes: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
