---
name: r-systempiper-workflow-execution
description: Use when running or reconstructing the QC step of the NEAT NGS pipeline
  on truncated H3K4me3 ChIPseq or RNAseq FASTQ test data, and per-cycle quality and
  read-distribution reports are required.
when_to_use_negative:
- Input is already aligned or post-QC (e.g. BAM files) — the QC module operates on
  decompressed FASTQ reads in the 'fatsq' folder.
- The pipeline has not yet run the 'unzip' module, so FASTQ files are still compressed
  and unrenamed in the project folder.
- A numeric quality threshold must be enforced as an acceptance criterion — the NEAT
  paper explicitly reports no numeric QC threshold, only output existence/completeness.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3577
tools:
- name: NEAT
  role: Pipeline framework that wraps the QC step as an automated, double-clickable
    module driven by the centralized Targets.txt file
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package on which the NEAT 'QC' module is implemented to compute
    the per-sample quality-control plots and metrics
  repo: https://github.com/tgirke/systemPipeR
- name: R
  role: Runtime language used by the QC module (via systemPipeR/Bioconductor) to generate
    the PDF quality reports
- name: Bioconductor
  role: Consortium providing the open-source R packages (including systemPipeR) that
    the QC module depends on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:4404c5da8dec4ea05c30fd79357a3d87210cd3d3fb5c8af07bc0bb7eb648584e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - systemPipeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# r-systempiper-workflow-execution

## Summary

Execute the quality-control step of the NEAT ChIPseq/RNAseq pipeline, which is implemented with the R/Bioconductor systemPipeR package and exposed to users as a double-clickable, automated module that produces human-readable PDF QC reports.

## When to use

Use when running or reconstructing the QC step of the NEAT NGS pipeline on truncated H3K4me3 ChIPseq or RNAseq FASTQ test data, and per-cycle quality and read-distribution reports are required.

## When NOT to use

- Input is already aligned or post-QC (e.g. BAM files) — the QC module operates on decompressed FASTQ reads in the 'fatsq' folder.
- The pipeline has not yet run the 'unzip' module, so FASTQ files are still compressed and unrenamed in the project folder.
- A numeric quality threshold must be enforced as an acceptance criterion — the NEAT paper explicitly reports no numeric QC threshold, only output existence/completeness.

## Inputs

- FASTQ files (truncated H3K4me3 ChIPseq or RNAseq test data from mouse embryonic stem cells)
- Targets.txt project file with OriFileName/FileName columns
- decompressed reads stored in the project's 'fatsq' folder

## Outputs

- PDF QC report graphs (per-cycle quality box plots, base proportion, relative k-mer diversity, read length and occurrence distribution, reads above quality cutoffs, mean quality distribution)
- QC output files for every registered sample (one report per sample, no missing samples)

## How to apply

Register the truncated FASTQ test files (bundled in the pschorderet/NEAT repository from mouse embryonic stem cells) in the project's centralized Targets.txt file, filling the OriFileName/FileName columns. Run the 'unzip' module logic first so files are decompressed, renamed and stored in the newly created 'fatsq' folder inside the project directory. Then execute the 'QC' module, which is built on the R systemPipeR package, to compute per-cycle quality box plots, base proportion, relative k-mer diversity, length and occurrence distribution of reads, number of reads above quality cutoffs and mean quality distribution. Finally export the results as human-readable report files (PDF graphs) matching the QC rendering in Fig. 3a of the NEAT article; the paper defines no numeric QC threshold, so output existence and completeness is the acceptance criterion.

## Related tools

- **NEAT** (Pipeline framework that wraps the QC step as an automated, double-clickable module driven by the centralized Targets.txt file) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package on which the NEAT 'QC' module is implemented to compute the per-sample quality-control plots and metrics) — https://github.com/tgirke/systemPipeR
- **R** (Runtime language used by the QC module (via systemPipeR/Bioconductor) to generate the PDF quality reports)
- **Bioconductor** (Consortium providing the open-source R packages (including systemPipeR) that the QC module depends on)

## Evaluation signals

- For every sample registered in Targets.txt, a QC report artifact exists with no missing-sample outputs.
- Each QC report contains all specified plot/metric categories: per-cycle quality box plots, base proportion, relative k-mer diversity, read length and occurrence distribution, reads above quality cutoffs, and mean quality distribution.
- Rendered QC graphs visually match the quality-control figure shown as Fig. 3a in the NEAT article.
- FASTQ inputs were correctly decompressed, renamed and located in the project's 'fatsq' folder before QC ran (i.e. the 'unzip' module completed first).
- The reports are human-readable PDFs suitable for archiving alongside other NEAT outputs, consistent with the small-sized-file sharing design of NEAT.

## Limitations

- The paper reports no numeric QC threshold; acceptance is judged only by existence and completeness of the specified output categories.
- NEAT is limited to Mac OSx (no Linux/Windows support stated) and its repository provides no changelog or stated version number.
- NEAT is still under development; bug reports must go to the authors, so the QC module behavior may change across commits.

## Evidence

- [methods] QC module built on systemPipeR: "The ‘QC’ module uses the R systemPipeR package"
- [intro] QC is one of NEAT's automated modules: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] systemPipeR repository URL: "URL https://github.com/tgirke/systemPipeR"
- [readme] NEAT can run without cluster terminal access: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
