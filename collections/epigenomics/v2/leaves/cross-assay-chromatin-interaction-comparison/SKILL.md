---
name: cross-assay-chromatin-interaction-comparison
description: Use when you have FASTQ data from related 3D genome mapping assays (ChIA-PET,
  HiChIP, PLAC-seq) and need to check whether their interaction calls agree with each
  other and with Hi-C.
when_to_use_negative:
- Data are already processed interaction call sets or .hic/.cool files rather than
  raw FASTQ reads
- The assay is none of ChIA-PET, HiChIP, or PLAC-seq and requires a different linker-filtering
  strategy not supported by the pseudolinker handling
- A quantitative numeric agreement threshold between assays is required, since the
  article validates comparability only qualitatively
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline performing linker filtering, read mapping, peak calling,
    loop calling, QC, and generation of contact map input files; supports HiChIP and
    PLAC-seq via pseudolinker handling
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: Juicebox
  role: Web-based 2D contact map viewer for the generated .hic files
- name: HiGlass
  role: Web-based viewer rendering the generated .cool contact maps
- name: BASIC Browser
  role: Loop browser for interactive, high-resolution visualization and cross-assay
    comparison of peaks and loops
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE, e.g., the dedup module retaining
    unique alignments (MAPQ ≥ 30) and nonredundant PETs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:c07dcd040490f74b84a175bb338bb410b387af2221008e7db413924bb79b5bd1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-assay-chromatin-interaction-comparison

## Summary

Process ChIA-PET, HiChIP, and PLAC-seq data with a single ChIA-PIPE run (handling their different junction sequences via pseudolinker handling) and compare the resulting chromatin interactions—2D contact maps and loop/domain calls—across assays and against Hi-C.

## When to use

Use when you have FASTQ data from related 3D genome mapping assays (ChIA-PET, HiChIP, PLAC-seq) and need to check whether their interaction calls agree with each other and with Hi-C.

## When NOT to use

- Data are already processed interaction call sets or .hic/.cool files rather than raw FASTQ reads
- The assay is none of ChIA-PET, HiChIP, or PLAC-seq and requires a different linker-filtering strategy not supported by the pseudolinker handling
- A quantitative numeric agreement threshold between assays is required, since the article validates comparability only qualitatively

## Inputs

- Paired-end FASTQ files from ChIA-PET, HiChIP, or PLAC-seq libraries (e.g., LDK0004-ds test FASTQs from Dropbox)
- ChIA-PIPE config file (e.g., example_config_file.sh) specifying parameters
- Reference genome for alignment

## Outputs

- .hic and .cool contact map files viewable in Juicebox and HiGlass
- Peak call outputs
- Loop call outputs
- 2D heat maps comparable across ChIA-PET, HiChIP, and PLAC-seq in the BASIC Browser
- Automated quality control reports per dataset

## How to apply

Clone ChIA-PIPE, install dependencies locally with local_install_chia_pipe_dependencies.sh into a dedicated dependency directory, and place paired-end FASTQ files (e.g., the LDK0004-ds test set) in a fastq/ subdirectory. Run chia_pipe-master/0.chia_pipe_shell.sh with example_config_file.sh (or submit 0.chia_pipe_hpc.pbs via qsub with the --conf flag on HPC), which executes linker filtering, read mapping, peak calling, loop calling, and automated quality control; ChIA-PIPE handles HiChIP and PLAC-seq despite different junction sequences between the two tags. Generate .hic and .cool files from the processed data so 2D contact maps render immediately in Juicebox and HiGlass. Inspect and compare the 2D heat maps across ChIA-PET, HiChIP, and PLAC-seq outputs in the BASIC Browser, and compare ChIA-PET loops and domains with those from Hi-C. Acceptance is qualitative: the pipeline completes and produces browser-ready .hic/.cool contact maps plus peak and loop call outputs for each assay.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline performing linker filtering, read mapping, peak calling, loop calling, QC, and generation of contact map input files; supports HiChIP and PLAC-seq via pseudolinker handling) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **Juicebox** (Web-based 2D contact map viewer for the generated .hic files)
- **HiGlass** (Web-based viewer rendering the generated .cool contact maps)
- **BASIC Browser** (Loop browser for interactive, high-resolution visualization and cross-assay comparison of peaks and loops)
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE, e.g., the dedup module retaining unique alignments (MAPQ ≥ 30) and nonredundant PETs) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- The pipeline run completes without errors on the test dataset (LDK0004-ds FASTQs) end-to-end
- Browser-ready .hic and .cool contact map files are produced and render in Juicebox/HiGlass
- Peak and loop call outputs are generated for each processed assay
- 2D heat maps for ChIA-PET, HiChIP, and PLAC-seq show comparable chromatin interactions when inspected side by side in the BASIC Browser
- ChIA-PET loops and domains qualitatively match loops and domains from Hi-C

## Limitations

- Cross-assay agreement is validated qualitatively via 2D heat maps and browser views; no numeric agreement threshold is reported
- HiChIP and PLAC-seq require pseudolinker handling because their junction sequences differ from ChIA-PET
- Older pipelines that peak-call without an input-control sample produce an abundance of false-positive peaks; this skill relies on ChIA-PIPE's input-control peak calling with stringent defaults
- No version or Docker image identifier for ChIA-PIPE itself is stated in the discussion section, and the reference genome version is unspecified there

## Evidence

- [results] Comparability finding: [evidence span withheld — anchor status: not_found]
