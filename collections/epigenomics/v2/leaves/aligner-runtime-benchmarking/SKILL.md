---
name: aligner-runtime-benchmarking
description: Use when an NGS workflow's alignment step dominates compute time (e.g.
  a 6-8 h baseline on 100 mammalian FASTQ files) and a decision is needed on whether
  swapping to HISAT2 or Rsubread within systemPipeR reduces it below ~3 h.
when_to_use_negative:
- The input is already aligned (e.g. BAM or a feature/counts table) — no alignment
  step exists to benchmark or swap.
- The analysis goal is alignment accuracy or sensitivity rather than compute-time
  reduction — this skill only compares wall-clock runtime.
- The workflow's compute time is dominated by a non-alignment step; the paper attributes
  most of the RNA-Seq workflow time to the Bowtie2/TopHat2 alignment step, so swapping
  aligners elsewhere gives little benefit.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: Workflow environment that loads the pre-configured RNA-Seq template and executes
    the modified alignment step; repo http://bioconductor.org/packages/devel/systemPipeR
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: HISAT2
  role: Faster RNA-Seq aligner substituted for the default Bowtie2/TopHat2 configuration
    to reduce runtime below 3 h
- name: Rsubread
  role: Alternative fast RNA-Seq aligner reported to achieve the same <3 h runtime
    for 100 FASTQ files
- name: Bowtie2
  role: Default aligner in the RNA-Seq template whose 6-8 h cluster runtime serves
    as the baseline in the comparison table
- name: TopHat2
  role: Default aligner configuration in the template replaced by HISAT2; contributes
    to the 6-8 h baseline runtime
- name: systemPipeRdata
  role: Companion package supplying the small sample FASTQ reads used to execute and
    time the modified workflow locally
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:7a7294f2380570ec7dc01eacefa4f10ea218dba69c6e489332cf8f173fcad34e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - HISAT2
  - R
  - Rsubread
  - TopHat2
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# aligner-runtime-benchmarking

## Summary

A method for benchmarking RNA-Seq alignment runtime inside a systemPipeR workflow by swapping the default Bowtie2/TopHat2 alignment step for a faster aligner (HISAT2 or Rsubread) and recording wall-clock timings for comparison.

## When to use

Use when an NGS workflow's alignment step dominates compute time (e.g. a 6-8 h baseline on 100 mammalian FASTQ files) and a decision is needed on whether swapping to HISAT2 or Rsubread within systemPipeR reduces it below ~3 h.

## When NOT to use

- The input is already aligned (e.g. BAM or a feature/counts table) — no alignment step exists to benchmark or swap.
- The analysis goal is alignment accuracy or sensitivity rather than compute-time reduction — this skill only compares wall-clock runtime.
- The workflow's compute time is dominated by a non-alignment step; the paper attributes most of the RNA-Seq workflow time to the Bowtie2/TopHat2 alignment step, so swapping aligners elsewhere gives little benefit.

## Inputs

- FASTQ short-read files (sample reads from systemPipeRdata for the local run; 100 mammalian FASTQ files for the reported baseline)
- systemPipeR pre-configured RNA-Seq workflow template/parameter files (Bowtie2/TopHat2 default alignment configuration)
- Reported baseline runtime for Bowtie2/TopHat2 (6-8 h on a cluster for 100 FASTQ files)

## Outputs

- Wall-clock runtime measurement of the HISAT2 alignment step within the systemPipeR workflow
- Two-row runtime comparison table written to runtime_alignment_comparison.csv (HISAT2 vs Bowtie2/TopHat2 baseline)

## How to apply

Initialize a fresh systemPipeR workflow project (spr_project) using the package's pre-configured RNA-Seq template, which by default invokes Bowtie2/TopHat2 for short read alignment of FASTQ inputs. Edit only the alignment step of the parameter file so it invokes HISAT2 instead of Bowtie2/TopHat2, leaving preprocessing and downstream RNA-Seq steps untouched so the runtime delta is attributable to the aligner. Execute the modified workflow on the small sample FASTQ reads shipped with the systemPipeRdata companion package, recording the wall-clock runtime of the HISAT2 alignment step. Assemble the HISAT2 timing alongside the paper's reported baseline (6-8 h on a cluster for 100 FASTQ files with Bowtie2/TopHat2 versus under 3 h for HISAT2/Rsubread) into a two-row comparison table written to runtime_alignment_comparison.csv.

## Related tools

- **systemPipeR** (Workflow environment that loads the pre-configured RNA-Seq template and executes the modified alignment step; repo http://bioconductor.org/packages/devel/systemPipeR) — http://bioconductor.org/packages/devel/systemPipeR
- **HISAT2** (Faster RNA-Seq aligner substituted for the default Bowtie2/TopHat2 configuration to reduce runtime below 3 h)
- **Rsubread** (Alternative fast RNA-Seq aligner reported to achieve the same <3 h runtime for 100 FASTQ files)
- **Bowtie2** (Default aligner in the RNA-Seq template whose 6-8 h cluster runtime serves as the baseline in the comparison table)
- **TopHat2** (Default aligner configuration in the template replaced by HISAT2; contributes to the 6-8 h baseline runtime)
- **systemPipeRdata** (Companion package supplying the small sample FASTQ reads used to execute and time the modified workflow locally)

## Evaluation signals

- runtime_alignment_comparison.csv exists and contains exactly two rows: the HISAT2 configuration and the Bowtie2/TopHat2 baseline.
- The HISAT2 row shows substantially lower runtime than the baseline, reproducing the reported pattern of <3 h versus 6-8 h for 100 FASTQ files.
- Only the alignment step of the workflow template was modified (Bowtie2/TopHat2 → HISAT2); all other RNA-Seq workflow steps are unchanged, so the runtime delta is attributable to the aligner.
- The HISAT2 alignment step completed successfully on the systemPipeRdata sample FASTQ reads, producing valid alignments recorded in the workflow project directory.
- The runtime measurement method (wall-clock of the alignment step) is recorded and reproducible within the spr_project workflow.

## Limitations

- The paper's runtime figures (6-8 h baseline vs <3 h for HISAT2/Rsubread) apply to 100 mammalian FASTQ files on a cluster; timings from the small sample reads are not directly comparable and must be reported alongside, not in place of, the baseline.
- Version requirements constrain reproducibility: R >=3.2 and Bioconductor >=3.2, which may limit portability to newer environments.
- Templates for additional NGS applications were still under development and not yet available at publication time, so this benchmarking pattern is grounded in the RNA-Seq template.

## Evidence

- [abstract] Pre-configured RNA-Seq templates and workflow uniformity: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [discussion] Command-line aligners (Bowtie2/TopHat2, HISAT2) are integrated as external tools via systemPipeR: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [abstract] systemPipeR supports executing workflows that combine R and command-line software: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [readme] README describes the project scope for building and executing workflows: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC)"
