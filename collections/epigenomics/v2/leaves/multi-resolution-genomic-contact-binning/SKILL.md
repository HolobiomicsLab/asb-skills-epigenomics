---
name: multi-resolution-genomic-contact-binning
description: Use when you have an analysis-ready, deduplicated ChIA-PET/HiChIP PET
  BAM (unique alignments, MAPQ >= 30, nonredundant PETs) and need browser-loadable
  2D contact maps at multiple resolutions for Juicebox.js or HiGlass.
when_to_use_negative:
- Input is raw FASTQ or linker-unfiltered PET reads — the contact matrix should be
  built only from the deduplicated BAM (unique, MAPQ >= 30, nonredundant PETs)
- The goal is single-resolution loop/peak calling rather than browser visualization
  of chromatin interaction heat maps
- A .hic or .cool contact map already exists and is loadable in the target browser
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3168
tools:
- name: ChIA-PIPE
  role: Automated pipeline that orchestrates the BAM-to-contact-map conversion and
    generates both .hic and .cool visualization files
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the 'dedup' module that produces the unique, MAPQ >= 30, nonredundant
    PET BAM used as input
  repo: https://github.com/cheehongsg/CPU
- name: Juicer
  role: Juicer Pre builds the binned contact matrix from merged_nodups.txt plus chromosome
    sizes and writes the .hic file
- name: cooler
  role: Converts the binned contact matrix to .cool (cooler HDF5) format for HiGlass
- name: Juicebox
  role: Web-based 2D contact map viewer that renders the .hic output
- name: HiGlass
  role: Web-based 2D contact map viewer that renders the .cool output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:0931795dea3ce1b1f571f4c10b8f8fd7669eb929a2cfb35d0793f796da6fd74e
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-resolution-genomic-contact-binning

## Summary

Convert the deduplicated PET alignment BAM from ChIA-PIPE's CPU 'dedup' module into multiscale 2D chromatin contact map files (.hic for Juicebox.js and .cool for HiGlass) so interaction heat maps are immediately viewable in web-based browsers.

## When to use

Use when you have an analysis-ready, deduplicated ChIA-PET/HiChIP PET BAM (unique alignments, MAPQ >= 30, nonredundant PETs) and need browser-loadable 2D contact maps at multiple resolutions for Juicebox.js or HiGlass.

## When NOT to use

- Input is raw FASTQ or linker-unfiltered PET reads — the contact matrix should be built only from the deduplicated BAM (unique, MAPQ >= 30, nonredundant PETs)
- The goal is single-resolution loop/peak calling rather than browser visualization of chromatin interaction heat maps
- A .hic or .cool contact map already exists and is loadable in the target browser

## Inputs

- Deduplicated PET alignment BAM from CPU 'dedup' (unique alignments, MAPQ >= 30, nonredundant PETs)
- Chromosome-size file for the reference genome assembly

## Outputs

- .hic multiscale contact matrix (10 resolutions from 2.5 Mb down to 1 kb) for Juicebox.js
- .cool (cooler HDF5) contact matrix for HiGlass
- Juicer-format merged_nodups.txt interaction records (intermediate)

## How to apply

Take the deduplicated PET BAM produced by the CPU 'dedup' module, which retained only unique alignments with MAPQ >= 30 and nonredundant PETs, and convert it into Juicer-format merged_nodups.txt interaction records. Run Juicer Pre on merged_nodups.txt together with chromosome-size information for the reference assembly to build the binned contact matrix. Write the .hic file at the 10 reported resolutions ranging from 2.5 Mb down to 1 kb so it renders directly in Juicebox, and convert the same matrix to cooler HDF5 (.cool) format so it is immediately usable by HiGlass. Finally, validate that both the .hic and .cool files were generated and are loadable in their respective browsers.

## Related tools

- **ChIA-PIPE** (Automated pipeline that orchestrates the BAM-to-contact-map conversion and generates both .hic and .cool visualization files) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the 'dedup' module that produces the unique, MAPQ >= 30, nonredundant PET BAM used as input) — https://github.com/cheehongsg/CPU
- **Juicer** (Juicer Pre builds the binned contact matrix from merged_nodups.txt plus chromosome sizes and writes the .hic file)
- **cooler** (Converts the binned contact matrix to .cool (cooler HDF5) format for HiGlass)
- **Juicebox** (Web-based 2D contact map viewer that renders the .hic output)
- **HiGlass** (Web-based 2D contact map viewer that renders the .cool output)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Both .hic and .cool files are generated and non-empty
- The .hic file is loadable and renders heat maps in Juicebox.js
- The .cool file is loadable and renders in HiGlass
- The .hic contains the expected 10 resolutions spanning 2.5 Mb down to 1 kb
- merged_nodups.txt contains only unique, nonredundant PET records consistent with the MAPQ >= 30 dedup filter

## Limitations

- Contact maps are built only from PETs passing the dedup filter (unique alignments, MAPQ >= 30, nonredundant); filtered-out PETs do not contribute
- Resolution list and binning depend on the chromosome-size information supplied for the reference assembly; the reference genome version is not specified in the discussed section
- ChIA-PIPE version 1.0 (November 26, 2018) is the documented release; no Docker image identifier is stated in this section

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) ≥ 30] and nonredundant PETs are retained by calling the CPU "dedup" module.: [evidence span withheld — anchor status: not_found]
