---
name: fastq-quality-control
description: Use when raw FASTQ files (e.g. truncated H3K4me3 ChIPseq or RNAseq reads
  from mouse embryonic stem cells) need pre-alignment quality assessment with per-cycle
  quality, base composition, k-mer and read-length reports.
when_to_use_negative:
- The reads are already aligned (BAM/GRanges) and downstream modules like 'map', 'filter'
  or 'peakcalling' are what is needed — QC operates on FASTQ via the unzip module.
- The analysis goal is post-mapping quality assessment or ChIP-rx normalization rather
  than pre-alignment FASTQ read quality.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: NEAT
  role: Framework exposing the QC step as a double-clickable automated module within
    its pipeline (unzip, rename, QC, chiprx, map, filter, peakcalling)
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package underlying the 'QC' module that computes the quality-control
    outputs and reports
  repo: https://github.com/tgirke/systemPipeR
- name: R
  role: Runtime environment for the QC module
- name: Bioconductor
  role: Package ecosystem providing systemPipeR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:e7d005547771bad905d777f4579c1054c73bae7bb7539bcbd23e8ffac992aecb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - systemPipeR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# fastq-quality-control

## Summary

Quality control of FASTQ reads as implemented in NEAT's 'QC' module, built on the R/Bioconductor systemPipeR package, producing per-sample human-readable PDF QC reports. This skill reproduces that QC step for ChIPseq/RNAseq FASTQ inputs.

## When to use

Use when raw FASTQ files (e.g. truncated H3K4me3 ChIPseq or RNAseq reads from mouse embryonic stem cells) need pre-alignment quality assessment with per-cycle quality, base composition, k-mer and read-length reports.

## When NOT to use

- The reads are already aligned (BAM/GRanges) and downstream modules like 'map', 'filter' or 'peakcalling' are what is needed — QC operates on FASTQ via the unzip module.
- The analysis goal is post-mapping quality assessment or ChIP-rx normalization rather than pre-alignment FASTQ read quality.

## Inputs

- FASTQ files (e.g. truncated H3K4me3 ChIPseq test fastq files bundled with NEAT)
- Targets.txt sample sheet with OriFileName/FileName columns (sample names, replicates, conditions)
- NEAT project directory structure

## Outputs

- Per-sample QC report files as human-readable PDF graphs matching Fig. 3a rendering
- QC metric categories: per-cycle quality box plots, base proportion, relative k-mer diversity, read length/occurrence distribution, number of reads above quality cutoffs, mean quality distribution

## How to apply

Register the FASTQ files in the project's centralized Targets.txt file using the OriFileName and FileName columns so each sample is tracked with its replicates and conditions. Run the 'unzip' module first to decompress, rename and store the files as fatsq files in a newly created folder inside the project directory. Then execute the 'QC' module, which uses the R systemPipeR package, to compute per cycle quality box plots, base proportion, relative k-mer diversity, length and occurrence distribution of reads, number of reads above quality cutoffs and mean quality distribution. Export the QC outputs as human-readable PDF graph reports, matching the rendering shown in Fig. 3a of the article. Because the paper states no numeric QC threshold, correctness is judged by existence and completeness of the specified output categories per sample, not by passing a cutoff.

## Related tools

- **NEAT** (Framework exposing the QC step as a double-clickable automated module within its pipeline (unzip, rename, QC, chiprx, map, filter, peakcalling)) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package underlying the 'QC' module that computes the quality-control outputs and reports) — https://github.com/tgirke/systemPipeR
- **R** (Runtime environment for the QC module)
- **Bioconductor** (Package ecosystem providing systemPipeR)

## Examples

```
Rscript -e 'library(systemPipeR); seeFastq(fq="fatsq/sample1.fq", batchsize=100000)'
```

## Evaluation signals

- For each test sample, the QC report artifact exists (no missing-sample outputs).
- Each report contains the specified plot/metric categories: per-cycle quality, base proportion, k-mer diversity, read length/occurrence distribution.
- Report rendering mirrors the quality-control figure shown in Fig. 3a of the article.
- The paper reports no numeric QC threshold, so existence and completeness of the specified output categories is the acceptance criterion — do not fail on numeric cutoffs.
- Inputs were correctly registered in Targets.txt (OriFileName/FileName) so every FASTQ is traced to a sample/replicate/condition.

## Limitations

- NEAT at the time of publication runs only on Mac OSx; no Linux/Windows support stated.
- No numeric QC threshold is defined by the paper; acceptance is by existence and completeness of output categories, not by quality cutoffs.
- NEAT is still under development per the README; bugs should be reported and the repository provides no changelog.
- QC requires the 'unzip' module to have run first, so raw compressed inputs must pass through unzip/rename before QC.

## Evidence

- [methods] The ‘QC’ module uses the R systemPipeR package: "The ‘QC’ module uses the R systemPipeR package"
- [methods] unzip module behavior: "The ‘unzip’ module will unzip, rename and store fatsq files in a newly created folder within the project folder."
- [intro] automated modules list: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] README pipeline composition: "NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline (i) Unzip and rename (ii) QC"
- [readme] human-readable outputs: "(i) Create human-readable data (pdf graphes, xls sheets, etc)"
