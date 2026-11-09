---
name: chromatin-peak-calling
description: Use when processing ChIA-PET (or HiChIP/PLAC-seq) libraries and protein-factor
  binding peaks must be called from self-ligation PETs to support loop calling, CCD
  calling, and enhancer-promoter annotation.
when_to_use_negative:
- The data are not chromatin-mapping reads (e.g., plain ChIP-seq or RNA-seq) with
  no PET linker structure, so self-ligation/interligation classification does not
  apply
- No input-control sample is available and false-positive peaks would invalidate the
  strict ChIA-PIPE peak-calling defaults
- The goal is alignment or linker filtering rather than peak detection — those are
  separate upstream ChIA-PIPE stages
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0607
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline executing the peak-calling stage (after linker filtering
    and read mapping) with input-control-based, stringent default peak calling
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities providing the 'dedup' module that retains unique alignments
    (MAPQ ≥ 30) and nonredundant PETs feeding peak calling
  repo: https://github.com/cheehongsg/CPU
- name: Juicebox
  role: 2D contact map viewer consuming .hic files generated alongside peak calls
    for visualization validation
- name: HiGlass
  role: 2D contact map viewer consuming .cool files generated alongside peak calls
    for visualization validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:549c4d0dc4449d5d0cf168f8af931b39af83fbb156d59a6a7670579e6e126e4c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatin-peak-calling

## Summary

Call protein-factor binding peaks from ChIA-PET self-ligation PETs as implemented in ChIA-PIPE, using uniquely aligned (MAPQ ≥ 30), nonredundant PETs and an input-control sample to suppress false positives, producing peak calls that anchor downstream loop and chromatin contact domain analysis.

## When to use

Use when processing ChIA-PET (or HiChIP/PLAC-seq) libraries and protein-factor binding peaks must be called from self-ligation PETs to support loop calling, CCD calling, and enhancer-promoter annotation.

## When NOT to use

- The data are not chromatin-mapping reads (e.g., plain ChIP-seq or RNA-seq) with no PET linker structure, so self-ligation/interligation classification does not apply
- No input-control sample is available and false-positive peaks would invalidate the strict ChIA-PIPE peak-calling defaults
- The goal is alignment or linker filtering rather than peak detection — those are separate upstream ChIA-PIPE stages

## Inputs

- ChIA-PET FASTQ read pairs (e.g., LDK0004-ds_*.fastq.gz test library)
- Uniquely aligned, deduplicated PETs (MAPQ ≥ 30) from ChIA-PIPE linker filtering and read mapping stages
- Input-control sequencing sample for peak calling
- ChIA-PIPE configuration file (e.g., example_config_file.sh) specifying library parameters

## Outputs

- Called protein-factor binding peaks (peak-call output files)
- Peak-supported loop anchors used by the loop-calling stage
- Automated quality control assessment for the dataset
- .hic and .cool visualization files for Juicebox and HiGlass contact map viewing

## How to apply

Run ChIA-PIPE's peak-calling stage after linker filtering and read mapping: retain only unique alignments with MAPQ ≥ 30 and nonredundant PETs via the CPU 'dedup' module, then classify PETs with a genomic span of ≤ 8 kb as self-ligation PETs, which are the substrate for peak detection. Call peaks against an input-control sample with stringent default parameter settings, because peak calling on very large datasets without an input control yields an abundance of false-positive peaks. Use the resulting peaks as anchor support for loop calling (loops require PET count ≥ 3 and peak support) and for CCD construction, which retains only loops with both anchors peak-supported and the top one-third by PET count (67th percentile cutoff). Peaks are also consumed by allele-specific resolution (reads overlapping phased SNPs, FDR Q ≤ 0.1) and enhancer-promoter loop annotation. Validate the run by confirming peak-call outputs alongside QC assessment files and .hic/.cool visualization files.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline executing the peak-calling stage (after linker filtering and read mapping) with input-control-based, stringent default peak calling) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities providing the 'dedup' module that retains unique alignments (MAPQ ≥ 30) and nonredundant PETs feeding peak calling) — https://github.com/cheehongsg/CPU
- **Juicebox** (2D contact map viewer consuming .hic files generated alongside peak calls for visualization validation)
- **HiGlass** (2D contact map viewer consuming .cool files generated alongside peak calls for visualization validation)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Pipeline completes and produces called peaks together with interaction loops, QC assessment outputs, and .hic/.cool files for the test library
- PETs entering peak calling are unique alignments with MAPQ ≥ 30 and nonredundant, as produced by the CPU dedup module
- Self-ligation PETs are correctly classified by the 8-kb genomic span cutoff before peak detection
- Peak calling was performed with an input-control sample and stringent default settings (no surge of false-positive peaks)
- Loop calls require PET count ≥ 3 plus peak support, so missing or weak peak calls should surface as unsupported loops

## Limitations

- Peak calling on very large datasets without an input-control sample results in an abundance of false-positive peaks
- Reference genome version for alignment is not specified in the article's discussion section, so genome-build consistency must be checked at configuration time
- ChIA-PIPE v1.0 (November 26, 2018) dependencies must be locally installed via local_install_chia_pipe_dependencies.sh before the pipeline will run

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) ≥ 30] and nonredundant PETs are retained by calling the CPU "dedup" module.: [evidence span withheld — anchor status: not_found]
