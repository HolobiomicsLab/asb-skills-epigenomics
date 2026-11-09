---
name: command-line-tool-parameterization
description: Use when an existing NGS workflow's default tool (e.g. Bowtie2/TopHat2
  alignment in systemPipeR) is too slow or unsuitable and you need to swap in an alternative
  command-line aligner by editing its parameter files and re-running the affected
  step.
when_to_use_negative:
- The analysis goal is not tool substitution or benchmarking — e.g. the default Bowtie2/TopHat2
  configuration already meets runtime and quality requirements
- The input is not FASTQ short-read data routed through a systemPipeR (or equivalent
  parameter-file-driven) workflow, since the parameter-editing approach depends on
  that configuration structure
- Only the downstream statistical steps (e.g. differential expression) need changes
  and the alignment step is untouched
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: systemPipeR
  role: Bioconductor workflow framework whose parameter files define each command-line
    step; edit the alignment step here to invoke a different aligner
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: HISAT2
  role: Faster RNA-Seq aligner swapped into the alignment step to reduce runtime below
    3 hours
- name: Bowtie2
  role: Default aligner in the RNA-Seq template; its runtime (6–8 h for 100 FASTQ
    files) serves as the baseline in the comparison table
- name: TopHat2
  role: Default alignment configuration in the RNA-Seq workflow template being replaced
    by HISAT2
- name: Rsubread
  role: Alternative fast RNA-Seq aligner that can replace the Bowtie2/TopHat2 step
- name: systemPipeRdata
  role: Companion package providing the small sample FASTQ reads used to test-run
    the modified workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:6e9e0419684dd724265f122fbdf305c026c1b8cd5ca181c707714ab30365c201
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# command-line-tool-parameterization

## Summary

Rewire and parameterize external command-line tools (e.g. swapping the default Bowtie2/TopHat2 alignment step of a systemPipeR RNA-Seq workflow to HISAT2 or Rsubread) by editing the workflow's parameter/configuration files, then execute and benchmark the modified step to compare wall-clock runtime against the published baseline.

## When to use

Use when an existing NGS workflow's default tool (e.g. Bowtie2/TopHat2 alignment in systemPipeR) is too slow or unsuitable and you need to swap in an alternative command-line aligner by editing its parameter files and re-running the affected step.

## When NOT to use

- The analysis goal is not tool substitution or benchmarking — e.g. the default Bowtie2/TopHat2 configuration already meets runtime and quality requirements
- The input is not FASTQ short-read data routed through a systemPipeR (or equivalent parameter-file-driven) workflow, since the parameter-editing approach depends on that configuration structure
- Only the downstream statistical steps (e.g. differential expression) need changes and the alignment step is untouched

## Inputs

- systemPipeR RNA-Seq workflow template with pre-configured parameter files (Bowtie2/TopHat2 alignment step)
- FASTQ files (100 mammalian RNA-Seq reads, or the small sample FASTQ reads shipped with systemPipeRdata)
- Published baseline runtime figures (6–8 h for Bowtie2/TopHat2 on 100 FASTQ files; <3 h for HISAT2/Rsubread)

## Outputs

- Modified workflow parameter file invoking HISAT2/Rsubread in place of Bowtie2/TopHat2
- Wall-clock runtime measurements for the HISAT2 alignment step
- Two-row runtime comparison table (runtime_alignment_comparison.csv)

## How to apply

Initialize a fresh systemPipeR workflow project (spr_project) from the package's pre-configured RNA-Seq template, which ships parameter files describing each command-line step. Edit the alignment step's parameter file so it invokes HISAT2 (or Rsubread) in place of the default Bowtie2/TopHat2 configuration, keeping index paths and FASTQ sample inputs unchanged. Execute the modified workflow — on the small sample FASTQ reads from the systemPipeRdata companion package first, then on the full 100 mammalian FASTQ files — recording wall-clock runtime of the alignment step. Rationale: the paper reports that the Bowtie2/TopHat2 alignment step accounts for most of the workflow's compute time (6–8 hours on a cluster for 100 FASTQ files), so replacing only that step with a faster aligner can reduce total compute to under 3 hours. Assemble the baseline and modified timings into a two-row runtime comparison table (e.g. runtime_alignment_comparison.csv) and confirm the speedup pattern reproduces the reported <3 h vs 6–8 h result.

## Related tools

- **systemPipeR** (Bioconductor workflow framework whose parameter files define each command-line step; edit the alignment step here to invoke a different aligner) — http://bioconductor.org/packages/devel/systemPipeR
- **HISAT2** (Faster RNA-Seq aligner swapped into the alignment step to reduce runtime below 3 hours)
- **Bowtie2** (Default aligner in the RNA-Seq template; its runtime (6–8 h for 100 FASTQ files) serves as the baseline in the comparison table)
- **TopHat2** (Default alignment configuration in the RNA-Seq workflow template being replaced by HISAT2)
- **Rsubread** (Alternative fast RNA-Seq aligner that can replace the Bowtie2/TopHat2 step)
- **systemPipeRdata** (Companion package providing the small sample FASTQ reads used to test-run the modified workflow)

## Examples

```
library(systemPipeR); library(systemPipeRdata); spr_project <- initProject(projPath='./rnaseq_hisat2', template='RNA-Seq'); # edit the alignment step parameter file to invoke HISAT2 in place of Bowtie2/TopHat2, then runWF() and record the alignment step runtime
```

## Evaluation signals

- The modified parameter file invokes HISAT2/Rsubread and no longer references Bowtie2/TopHat2 for the alignment step
- The workflow completes end-to-end on the systemPipeRdata sample FASTQ reads without errors after the alignment-step edit
- runtime_alignment_comparison.csv contains two rows (baseline and HISAT2 configuration) with the HISAT2 runtime substantially lower than the 6–8 h Bowtie2/TopHat2 baseline
- The HISAT2 configuration runtime falls below the reported 3-hour threshold when scaled to 100 FASTQ files
- Alignment output files for the new aligner are produced in the expected workflow step location with consistent naming

## Limitations

- Runtime figures are workload-dependent: the 6–8 h vs <3 h comparison is reported for 100 mammalian FASTQ files on a cluster and may not transfer to other sample counts or hardware
- Older version constraints may limit reproducibility with newer environments: the paper's requirements are R version ≥3.2 and Bioconductor version ≥3.2
- Templates for additional NGS applications were still under development and not yet available at publication time

## Evidence

- [abstract] abstract: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [discussion] discussion: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [abstract] pre-configured workflows: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [readme] readme: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems, while simultaneously generating publication-quality analysis"
