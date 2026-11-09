---
name: chia-pet-contact-matrix-generation
description: Use when a deduplicated ChIA-PET PET alignment BAM (unique, MAPQ >= 30,
  nonredundant PETs from CPU dedup) must be turned into .hic and/or .cool contact
  maps for Juicebox.js or HiGlass visualization.
when_to_use_negative:
- Input is raw FASTQ or a pre-dedup BAM — the skill expects the analysis-ready BAM
  already filtered to unique, MAPQ >= 30, nonredundant PETs by CPU dedup.
- The goal is 3D genome viewer rendering of peaks/loops rather than 2D contact maps
  — ChIA-PIPE provides a separate dockerized visualization tool (BASIC Browser) for
  loop exploration.
- Data are single-end or non-PET chromatin interaction reads that cannot be represented
  as paired-end tag interaction records for Juicer Pre.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Automated pipeline that orchestrates the conversion of the dedup BAM into
    merged_nodups.txt and launches the contact matrix generation
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: Juicer
  role: Juicer Pre builds the binned contact matrix from merged_nodups.txt plus chromosome
    sizes and writes the .hic file
- name: cooler
  role: Converts the binned contact matrix to .cool HDF5 format for HiGlass
- name: Juicebox
  role: Web-based 2D contact map viewer that renders the .hic file
- name: HiGlass
  role: Web-based 2D contact map viewer that renders the .cool file
- name: CPU (ChIA-PET Utilities)
  role: Its 'dedup' module produces the upstream analysis-ready BAM of unique, nonredundant
    PETs that feeds contact matrix generation
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:c6e42af08587dda584ba75d09a901210e7ba119ff8dedd07a098de8d9a215ea2
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chia-pet-contact-matrix-generation

## Summary

Converts the analysis-ready deduplicated ChIA-PET alignment BAM into binned 2D contact matrix files — a .hic for Juicebox.js and a .cool for HiGlass — so chromatin interaction heat maps are immediately browser-renderable after ChIA-PIPE completes.

## When to use

Use when a deduplicated ChIA-PET PET alignment BAM (unique, MAPQ >= 30, nonredundant PETs from CPU dedup) must be turned into .hic and/or .cool contact maps for Juicebox.js or HiGlass visualization.

## When NOT to use

- Input is raw FASTQ or a pre-dedup BAM — the skill expects the analysis-ready BAM already filtered to unique, MAPQ >= 30, nonredundant PETs by CPU dedup.
- The goal is 3D genome viewer rendering of peaks/loops rather than 2D contact maps — ChIA-PIPE provides a separate dockerized visualization tool (BASIC Browser) for loop exploration.
- Data are single-end or non-PET chromatin interaction reads that cannot be represented as paired-end tag interaction records for Juicer Pre.

## Inputs

- Deduplicated PET alignment BAM from the CPU 'dedup' module (unique alignments, MAPQ >= 30, nonredundant PETs)
- Chromosome-size file for the reference assembly
- Reference assembly information (e.g., hg19)

## Outputs

- merged_nodups.txt (Juicer-format interaction records)
- Binned contact matrix (.hic) at 10 resolutions from 2.5 Mb down to 1 kb for Juicebox.js
- Contact matrix in cooler HDF5 (.cool) format for HiGlass

## How to apply

Start from the deduplicated PET alignment BAM produced by the CPU 'dedup' module, which retains only unique alignments with MAPQ >= 30 and nonredundant PETs. Convert these alignments into Juicer-format merged_nodups.txt interaction records, then run Juicer Pre on merged_nodups.txt together with chromosome-size information for the reference assembly to build the binned contact matrix. Write the .hic file at the 10 reported resolutions ranging from 2.5 Mb down to 1 kb so it renders directly in Juicebox.js. Convert the same binned matrix to cooler HDF5 (.cool) format so it is immediately usable by HiGlass. Finally, validate that both files were generated and are loadable in their respective browsers.

## Related tools

- **ChIA-PIPE** (Automated pipeline that orchestrates the conversion of the dedup BAM into merged_nodups.txt and launches the contact matrix generation) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **Juicer** (Juicer Pre builds the binned contact matrix from merged_nodups.txt plus chromosome sizes and writes the .hic file)
- **cooler** (Converts the binned contact matrix to .cool HDF5 format for HiGlass)
- **Juicebox** (Web-based 2D contact map viewer that renders the .hic file)
- **HiGlass** (Web-based 2D contact map viewer that renders the .cool file)
- **CPU (ChIA-PET Utilities)** (Its 'dedup' module produces the upstream analysis-ready BAM of unique, nonredundant PETs that feeds contact matrix generation) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Both a .hic and a .cool file are present in the pipeline output directory after the run completes.
- The .hic file contains the full set of 10 resolutions, spanning 2.5 Mb down to 1 kb.
- merged_nodups.txt exists in valid Juicer format and matches the deduplicated PET content of the input BAM.
- The .hic file loads and renders in Juicebox.js and the .cool file loads and renders in HiGlass.
- Contact matrix bins are consistent with the chromosome-size file for the reference assembly.

## Limitations

- The skill is downstream of quality filtering: only PETs surviving CPU dedup (unique, MAPQ >= 30, nonredundant) contribute to the matrix, so aggressive filtering will shrink the map.
- Correct chromosome-size information for the exact reference assembly must be supplied to Juicer Pre; a mismatched assembly will produce misbinned maps.
- The article/README context does not state a ChIA-PIPE version beyond v1.0 (November 26, 2018) or a Docker image identifier for the contact matrix step itself.

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) >= 30] and nonredundant PETs are retained by calling the CPU "dedup" module.: [evidence span withheld — anchor status: not_found]
