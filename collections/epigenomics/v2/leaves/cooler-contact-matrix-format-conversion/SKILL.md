---
name: cooler-contact-matrix-format-conversion
description: Use when a deduplicated ChIA-PET/HIChIP PET BAM (unique alignments, MAPQ
  ≥ 30, nonredundant PETs) must be turned into .hic and/or .cool files for visualization
  in Juicebox.js or HiGlass.
when_to_use_negative:
- The input is raw FASTQ or linker-unfiltered reads — upstream linker filtering, mapping,
  and dedup (MAPQ >= 30) must run first
- The goal is peak/loop calling or CCD identification rather than 2D contact-map visualization
- The target viewer is a non-.hic/.cool tool such as WashU Epigenome Browser or 3D
  Genome Browser formats not covered by this conversion
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: ChIA-PIPE
  role: Automated pipeline that orchestrates the conversion of the deduplicated PET
    BAM into .hic and .cool contact map files
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Provides the 'dedup' module that produces the nonredundant MAPQ >= 30 PET
    BAM used as conversion input
  repo: https://github.com/cheehongsg/CPU
- name: Juicer
  role: Juicer Pre converts merged_nodups.txt plus chromosome sizes into the binned
    .hic contact matrix
- name: cooler
  role: Converts the same contact matrix to the .cool HDF5 format for HiGlass
- name: Juicebox.js
  role: Web-based viewer in which the generated .hic contact map is rendered
- name: HiGlass
  role: Web-based viewer in which the generated .cool contact map is rendered
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:2f2c436f0db95b32cd834d98d1dad7f4a3b054b29870cbd8fe29136bc887eed6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  - Juicebox.js
  - Juicer
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cooler-contact-matrix-format-conversion

## Summary

Converts ChIA-PIPE's analysis-ready, deduplicated PET alignment BAM into multiscale 2D contact map files — a .hic for Juicebox.js and a .cool (cooler HDF5) for HiGlass — so chromatin interaction heat maps are immediately browser-renderable.

## When to use

Use when a deduplicated ChIA-PET/HIChIP PET BAM (unique alignments, MAPQ ≥ 30, nonredundant PETs) must be turned into .hic and/or .cool files for visualization in Juicebox.js or HiGlass.

## When NOT to use

- The input is raw FASTQ or linker-unfiltered reads — upstream linker filtering, mapping, and dedup (MAPQ >= 30) must run first
- The goal is peak/loop calling or CCD identification rather than 2D contact-map visualization
- The target viewer is a non-.hic/.cool tool such as WashU Epigenome Browser or 3D Genome Browser formats not covered by this conversion

## Inputs

- Analysis-ready deduplicated ChIA-PET PET alignment BAM (unique alignments, MAPQ >= 30, nonredundant PETs from the CPU 'dedup' module)
- Chromosome-size file for the reference genome assembly

## Outputs

- merged_nodups.txt (Juicer-format interaction records)
- .hic multiscale contact matrix at 10 resolutions from 2.5 Mb to 1 kb (for Juicebox.js)
- .cool cooler HDF5 contact matrix (for HiGlass)

## How to apply

Take the deduplicated PET alignment BAM produced by the CPU 'dedup' module (unique alignments, MAPQ ≥ 30, nonredundant PETs) and convert it to Juicer-format merged_nodups.txt interaction records. Run Juicer Pre on merged_nodups.txt together with chromosome-size information for the reference assembly to build the binned contact matrix, and write the .hic file at 10 reported resolutions ranging from 2.5 Mb down to 1 kb so it renders directly in Juicebox. Convert the same contact matrix to cooler HDF5 (.cool) format for HiGlass. Finally, verify that both the .hic and .cool files were generated and are loadable in Juicebox.js and HiGlass respectively.

## Related tools

- **ChIA-PIPE** (Automated pipeline that orchestrates the conversion of the deduplicated PET BAM into .hic and .cool contact map files) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Provides the 'dedup' module that produces the nonredundant MAPQ >= 30 PET BAM used as conversion input) — https://github.com/cheehongsg/CPU
- **Juicer** (Juicer Pre converts merged_nodups.txt plus chromosome sizes into the binned .hic contact matrix)
- **cooler** (Converts the same contact matrix to the .cool HDF5 format for HiGlass)
- **Juicebox.js** (Web-based viewer in which the generated .hic contact map is rendered)
- **HiGlass** (Web-based viewer in which the generated .cool contact map is rendered)

## Examples

```
git clone https://github.com/TheJacksonLaboratory/ChIA-PIPE.git && bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Both a .hic file and a .cool file are generated for the library and neither is empty
- The .hic matrix contains all 10 reported resolutions spanning 2.5 Mb down to 1 kb
- The .hic loads and renders in Juicebox.js and the .cool loads and renders in HiGlass
- merged_nodups.txt follows Juicer-format interaction-record schema and was built only from unique, MAPQ >= 30, nonredundant PETs
- Binning used chromosome-size information matching the reference assembly used for alignment

## Limitations

- The contact matrix resolution range is fixed to the 10 reported resolutions (2.5 Mb to 1 kb); other resolutions are not described in this context
- Only PETs retained by the CPU 'dedup' module (unique, MAPQ >= 30, nonredundant) feed the contact map, so low-MAPQ or redundant PETs are excluded
- Correct rendering depends on chromosome-size information matching the reference assembly used for alignment

## Evidence

- [discussion] The unique alignments [MAPQ >= 30] and nonredundant PETs are retained by calling the CPU dedup module: [evidence span withheld — anchor status: not_found]
