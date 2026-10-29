---
name: contact-map-visualization-file-generation
description: Use when a deduplicated PET alignment BAM (unique, MAPQ >= 30, nonredundant
  PETs) is available and the goal is to produce .hic and .cool 2D contact map files
  for Juicebox.js or HiGlass visualization.
when_to_use_negative:
- The input is not an analysis-ready deduplicated PET BAM — e.g. raw FASTQ or linker-filtered
  reads have not yet been mapped and deduplicated
- The target viewer is neither Juicebox/Juicebox.js nor HiGlass (e.g. WashU Epigenome
  Browser or 3D Genome Browser), since this step specifically generates .hic and .cool
  formats
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: ChIA-PIPE
  role: Automatically generates the 2D contact map input files (.hic and .cool) as
    part of the pipeline run
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: Juicer
  role: Juicer Pre builds the binned contact matrix from merged_nodups.txt interaction
    records
- name: Juicebox
  role: Renders the generated .hic contact map for 2D visualization of chromatin interactions
- name: HiGlass
  role: Renders the generated .cool (cooler HDF5) contact map in the browser
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:9ac0b6a285b9bce989ef03c6d4533cf53438c11b7e035b4a3628bc2179c92a81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  - Juicer
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# contact-map-visualization-file-generation

## Summary

Convert an analysis-ready ChIA-PET PET alignment BAM into multiscale 2D contact map files (.hic for Juicebox.js and .cool for HiGlass) so chromatin interaction heat maps are immediately renderable in browser-based viewers.

## When to use

Use when a deduplicated PET alignment BAM (unique, MAPQ >= 30, nonredundant PETs) is available and the goal is to produce .hic and .cool 2D contact map files for Juicebox.js or HiGlass visualization.

## When NOT to use

- The input is not an analysis-ready deduplicated PET BAM — e.g. raw FASTQ or linker-filtered reads have not yet been mapped and deduplicated
- The target viewer is neither Juicebox/Juicebox.js nor HiGlass (e.g. WashU Epigenome Browser or 3D Genome Browser), since this step specifically generates .hic and .cool formats

## Inputs

- Deduplicated PET alignment BAM (unique alignments, MAPQ >= 30, nonredundant PETs) from the CPU 'dedup' module
- Chromosome-size information for the reference assembly

## Outputs

- .hic contact matrix at 10 resolutions from 2.5 Mb down to 1 kb (Juicebox/Juicebox.js)
- .cool (cooler HDF5) contact matrix (HiGlass)

## How to apply

Start from the deduplicated PET alignment BAM produced by the CPU 'dedup' module, which retains unique alignments with MAPQ >= 30 and nonredundant PETs. Convert these alignments into Juicer-format merged_nodups.txt interaction records, then run Juicer Pre on merged_nodups.txt together with chromosome-size information for the reference assembly to build the binned contact matrix. Write the .hic file at the 10 reported resolutions spanning 2.5 Mb down to 1 kb so it renders directly in Juicebox, and convert the same matrix to cooler HDF5 (.cool) format for HiGlass. ChIA-PIPE automates this generation as part of the pipeline so that visualization files are usable immediately upon completion.

## Related tools

- **ChIA-PIPE** (Automatically generates the 2D contact map input files (.hic and .cool) as part of the pipeline run) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **Juicer** (Juicer Pre builds the binned contact matrix from merged_nodups.txt interaction records)
- **Juicebox** (Renders the generated .hic contact map for 2D visualization of chromatin interactions)
- **HiGlass** (Renders the generated .cool (cooler HDF5) contact map in the browser)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Both a .hic and a .cool file are generated for the dataset
- The .hic file loads and renders in Juicebox
- The .cool file loads and renders in HiGlass
- The .hic matrix contains the expected multiscale bins, spanning resolutions from 2.5 Mb down to 1 kb
- merged_nodups.txt interaction records were produced from the deduplicated PET BAM before matrix construction

## Limitations

- Peak calling on very large datasets without an input-control sample yields an abundance of false-positive peaks, which would propagate into misleading loop-supported contact maps
- The pipeline was designed for ChIA-PET scale datasets of hundreds of millions of PET reads (e.g. ~300 million read pairs requiring 20 threads and 60 Gb RAM); smaller or unrelated assay types need configuration

## Evidence

- [other] ChIA-PIPE automatically generates 2D contact map files for both browser-based tools: a .hic file for Juicebox.js and a .cool file for HiGlass.: [evidence span withheld — anchor status: not_found]
- [discussion] The unique alignments [MAPQ (mapping quality) ≥ 30] and nonredundant PETs are retained by calling the CPU "dedup" module.: [evidence span withheld — anchor status: not_found]
