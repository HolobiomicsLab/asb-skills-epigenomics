---
name: ngs-read-quality-visualization
description: Use when raw FASTQ reads (e.g. truncated H3K4me3 ChIPseq or RNAseq data
  from mouse embryonic stem cells) have been unzipped and registered in Targets.txt
  and a per-sample sequencing-quality report must be generated before mapping.
when_to_use_negative:
- The pipeline stage is downstream of QC — use the 'map', 'filter', 'peakcalling',
  or 'GRanges' modules instead of re-running quality visualization on already-mapped
  BAM/GRanges data
- A quantitative QC threshold decision is required (e.g. discard/keep calls) — the
  article reports no numeric QC threshold, only existence and completeness of output
  categories
- Non-NGS data or platforms outside NEAT's ChIPseq/RNAseq scope — this module is bound
  to NEAT's Targets.txt project structure and systemPipeR workflow
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NEAT
  role: Framework providing the double-clickable automated 'QC' module and the centralized
    Targets.txt project file driving it
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package underlying the QC module, computing the quality plots
    and metrics
  repo: https://github.com/tgirke/systemPipeR
- name: R
  role: Runtime language for the QC module implementation
- name: Bioconductor
  role: Package ecosystem supplying the R packages (incl. systemPipeR) used by the
    QC module
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:b41a824e86a44734fd96fe0aa5041a6517d004b0b835e39e687a729da2c7515d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - systemPipeR
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ngs-read-quality-visualization

## Summary

Runs the QC step of NEAT's automated ChIPseq/RNAseq pipeline, built on the R systemPipeR package, to compute per-cycle quality box plots, base proportions, relative k-mer diversity, read length/occurrence distributions, reads above quality cutoffs, and mean quality distribution as human-readable pdf reports (as rendered in Fig. 3a).

## When to use

Use when raw FASTQ reads (e.g. truncated H3K4me3 ChIPseq or RNAseq data from mouse embryonic stem cells) have been unzipped and registered in Targets.txt and a per-sample sequencing-quality report must be generated before mapping.

## When NOT to use

- The pipeline stage is downstream of QC — use the 'map', 'filter', 'peakcalling', or 'GRanges' modules instead of re-running quality visualization on already-mapped BAM/GRanges data
- A quantitative QC threshold decision is required (e.g. discard/keep calls) — the article reports no numeric QC threshold, only existence and completeness of output categories
- Non-NGS data or platforms outside NEAT's ChIPseq/RNAseq scope — this module is bound to NEAT's Targets.txt project structure and systemPipeR workflow

## Inputs

- FASTQ files (truncated H3K4me3 ChIPseq or RNAseq test data from mouse embryonic stem cells) stored in the 'fatsq' folder after the unzip module
- Targets.txt sample sheet with OriFileName/FileName columns listing samples and replicates

## Outputs

- Per-sample QC pdf report graphs (per-cycle quality box plots, base proportion, relative k-mer diversity, read length and occurrence distributions)
- QC metrics: number of reads above quality cutoffs and mean quality distribution per sample

## How to apply

First run the 'unzip' module so fastq files are decompressed, renamed and stored in the newly created 'fatsq' folder inside the project directory, with samples registered in the centralized Targets.txt (OriFileName/FileName columns). Then execute the double-clickable 'QC' module, which uses the R systemPipeR package to compute the specified quality-control outputs: per cycle quality box plots, base proportion, relative k-mer diversity, length and occurrence distribution of reads, number of reads above quality cutoffs and mean quality distribution. The module exports results as human-readable report files (pdf graphs) matching the quality-control rendering shown in Fig. 3a of the article. Because the paper reports no numeric QC threshold, correctness is judged by existence and completeness of the specified output categories for every registered sample, not by pass/fail cutoffs.

## Related tools

- **NEAT** (Framework providing the double-clickable automated 'QC' module and the centralized Targets.txt project file driving it) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package underlying the QC module, computing the quality plots and metrics) — https://github.com/tgirke/systemPipeR
- **R** (Runtime language for the QC module implementation)
- **Bioconductor** (Package ecosystem supplying the R packages (incl. systemPipeR) used by the QC module)

## Evaluation signals

- For each sample in Targets.txt, QC report artifacts exist in the project directory (no missing-sample outputs)
- Each QC report contains the specified plot/metric categories: per-cycle quality, base proportion, relative k-mer diversity, read length/occurrence distribution, reads above quality cutoffs, mean quality distribution
- Reports are human-readable pdf graphs matching the quality-control rendering shown in Fig. 3a of the article
- Inputs were produced by the unzip module: fastq files present in the newly created 'fatsq' folder inside the project folder
- Since the paper reports no numeric QC threshold, acceptance is existence and completeness of output categories, not threshold-based pass/fail

## Limitations

- The paper reports no numeric QC threshold, so this skill cannot make quantitative keep/discard decisions on read quality
- NEAT (and hence this QC module) is limited to Mac OSx per the discussion section; no Linux/Windows support stated
- NEAT is still under development — the README directs bug reports to the authors
- Test datasets are truncated H3K4me3 ChIPseq or RNAseq data from mouse embryonic stem cells; results on full-size or other datasets are not benchmarked in the article

## Evidence

- [methods] The ‘QC’ module uses the R systemPipeR package: "The ‘QC’ module uses the R systemPipeR package"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [readme] NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications.: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
- [results] As this work presents a 'pipeline', tangible results are in the form of outputs (Fig. 3). Supporting arguments are in-line.: "As this work presents a 'pipeline', tangible results are in the form of outputs (Fig. 3). Supporting arguments are in-line."
