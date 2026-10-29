---
name: juicer-pre-hic-conversion
description: Use when a deduplicated PET alignment BAM (unique alignments, MAPQ >=
  30, nonredundant PETs) must be rendered as browser-viewable chromatin interaction
  heat maps in Juicebox.js or HiGlass.
when_to_use_negative:
- Input is raw FASTQ or a non-deduplicated alignment BAM — the conversion expects
  the analysis-ready BAM of nonredundant PETs (MAPQ >= 30) already produced by CPU
  'dedup'
- The goal is loop/peak calling or chromatin contact domain (CCD) calling rather than
  contact-map visualization
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline that orchestrates the conversion of the analysis-ready
    BAM into .hic and .cool visualization files
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: Juicer
  role: Juicer Pre builds the binned contact matrix from merged_nodups.txt and chromosome
    sizes and writes the .hic file
- name: cooler
  role: Converts the contact matrix to .cool (HDF5) format for HiGlass
- name: Juicebox
  role: Web-based 2D contact map viewer that renders the generated .hic file
- name: HiGlass
  role: Browser-based 2D contact map viewer that renders the generated .cool file
- name: CPU (ChIA-PET Utilities)
  role: Its 'dedup' module produces the unique-alignment (MAPQ >= 30), nonredundant
    PET BAM that is the input to the conversion
  repo: github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:23b377c76a042f937d036d94178892b7861a8d2b6ae482c8a36863641cb80da2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  - Juicer
  - cooler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# juicer-pre-hic-conversion

## Summary

Converts the deduplicated, analysis-ready ChIA-PET BAM from the CPU 'dedup' module into multiscale 2D contact map files — a .hic file for Juicebox.js and a .cool (HDF5) file for HiGlass — via Juicer-format merged_nodups.txt records and Juicer Pre.

## When to use

Use when a deduplicated PET alignment BAM (unique alignments, MAPQ >= 30, nonredundant PETs) must be rendered as browser-viewable chromatin interaction heat maps in Juicebox.js or HiGlass.

## When NOT to use

- Input is raw FASTQ or a non-deduplicated alignment BAM — the conversion expects the analysis-ready BAM of nonredundant PETs (MAPQ >= 30) already produced by CPU 'dedup'
- The goal is loop/peak calling or chromatin contact domain (CCD) calling rather than contact-map visualization

## Inputs

- Deduplicated PET alignment BAM from the CPU 'dedup' module (unique alignments, MAPQ >= 30, nonredundant PETs)
- Chromosome-size file for the reference genome assembly

## Outputs

- Juicer-format merged_nodups.txt interaction records (intermediate)
- Binned .hic contact matrix at 10 resolutions from 2.5 Mb down to 1 kb (for Juicebox.js)
- .cool (cooler HDF5) contact matrix (for HiGlass)

## How to apply

Take the deduplicated PET alignment BAM produced by the CPU 'dedup' module (unique alignments with MAPQ >= 30 and nonredundant PETs) and convert it into Juicer-format merged_nodups.txt interaction records. Run Juicer Pre on merged_nodups.txt together with chromosome-size information for the reference assembly to build the binned contact matrix. Write the .hic contact matrix at the 10 reported resolutions ranging from 2.5 Mb down to 1 kb so it renders directly in Juicebox. Convert the same matrix to .cool (cooler HDF5) format so it is immediately usable by HiGlass. Validate that both the .hic and .cool files are generated and loadable/renderable in Juicebox and HiGlass, respectively.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline that orchestrates the conversion of the analysis-ready BAM into .hic and .cool visualization files) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **Juicer** (Juicer Pre builds the binned contact matrix from merged_nodups.txt and chromosome sizes and writes the .hic file)
- **cooler** (Converts the contact matrix to .cool (HDF5) format for HiGlass)
- **Juicebox** (Web-based 2D contact map viewer that renders the generated .hic file)
- **HiGlass** (Browser-based 2D contact map viewer that renders the generated .cool file)
- **CPU (ChIA-PET Utilities)** (Its 'dedup' module produces the unique-alignment (MAPQ >= 30), nonredundant PET BAM that is the input to the conversion) — github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Both a .hic file and a .cool file are generated by the end of the pipeline run
- The .hic file loads and renders in Juicebox.js with no errors
- The .cool file loads and renders in HiGlass with no errors
- The .hic contact matrix contains the 10 reported resolutions, from 2.5 Mb down to 1 kb
- merged_nodups.txt interaction records correspond to unique alignments (MAPQ >= 30) and nonredundant PETs from the dedup BAM

## Limitations

- Chromosome-size information for the correct reference assembly must be supplied; the article does not specify the reference genome version in the discussion section
- The conversion depends on the CPU 'dedup' module having run first; MAPQ >= 30 filtering is applied upstream
- The README documents ChIA-PIPE version 1.0 (November 26, 2018); no Docker image identifier for the visualization steps is stated in this section

## Evidence

- [discussion] CPU dedup retains unique, nonredundant PETs: [evidence span withheld — anchor status: not_found]
