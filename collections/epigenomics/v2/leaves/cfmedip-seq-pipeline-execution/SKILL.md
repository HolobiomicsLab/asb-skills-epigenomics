---
name: cfmedip-seq-pipeline-execution
description: Use when raw cfMeDIP-seq or (cf)MeDIP-seq FASTQ files (single-end or
  paired-end, with or without spike-in/UMI sequences) need automated QC, alignment,
  methylation quantification, and multi-sample aggregation into tab-delimited bin-sample
  matrices.
when_to_use_negative:
- Input data is not MeDIP-seq/cfMeDIP-seq enrichment data (e.g. whole-genome bisulfite
  sequencing or array methylation), since MEDIPIPE's QC and quantification models
  (MEDIPS/QSEA/MEDStrand) assume MeDIP enrichment profiles
- Data is already aligned and QC'd (BAM-level inputs) and only downstream statistical
  modelling is needed — MEDIPIPE starts from raw FASTQ files
- Single-sample exploratory analysis where the aggregation and group/batch-level QC
  reporting machinery is unnecessary overhead
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: Snakemake
  role: Workflow engine that orchestrates all pipeline steps and automatically deploys
    containerized execution environments
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Environment manager for the core MEDIPIPE environment and per-tool environments
- name: Mamba
  role: Recommended fast Conda reimplementation for installing the pipeline environment
  repo: https://github.com/mamba-org/mamba
- name: SLURM
  role: Cluster engine supported for parallel per-sample job submission via the sbatch
    Snakefile template
- name: Trim Galore
  role: Adapter/quality trimming (v0.6.7) of raw FASTQ reads
- name: FastQC
  role: Per-sample read QC (v0.11.9) after trimming
- name: UMI-tools
  role: Barcode extraction and deduplication (v1.0.1) when UMIs are present
- name: BWA-MEM
  role: Alignment of trimmed reads to the reference genome (v0.7.17)
- name: SAMtools
  role: Alignment filtering and duplicate removal (v1.17)
- name: Picard
  role: CollectInsertSizeMetrics and fragmentation size ratios in consecutive 1 and
    5 Mb windows (v2.26.6)
- name: MEDIPS
  role: Methylation quantification with linear regression model (v1.46.0) plus saturation/coverage/enrichment
    QC
- name: QSEA
  role: Methylation quantification with sigmoidal model (v1.20.0)
- name: MEDStrand
  role: Stranded sigmoid-model methylation quantification
- name: MultiQC
  role: Aggregation of per-sample QC into a summarized interactive HTML report (v1.0.dev0)
- name: Tabix
  role: Indexing of original and filtered aggregated methylation TXT files (v1.17)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:09c33efa69cb82e1f89ca42f946f271b23682ecb57449a4356e9299ea2b3af6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - BWA-MEM
  - Conda
  - FastQC
  - MEDIPS
  - MEDStrand
  - Mamba
  - MultiQC
  - Picard
  - QSEA
  - SAMtools
  - SLURM
  - Snakemake
  - Tabix
  - Trim Galore
  - UMI-tools
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cfmedip-seq-pipeline-execution

## Summary

Execute MEDIPIPE, a Snakemake-based end-to-end pipeline that takes cfMeDIP-seq raw FASTQ files through QC, alignment, methylation quantification, and aggregation, producing aggregated bin-sample methylation matrices and summarized QC reports. It automates per-tool environment deployment via Conda and scales across cluster engines such as SLURM.

## When to use

Use when raw cfMeDIP-seq or (cf)MeDIP-seq FASTQ files (single-end or paired-end, with or without spike-in/UMI sequences) need automated QC, alignment, methylation quantification, and multi-sample aggregation into tab-delimited bin-sample matrices.

## When NOT to use

- Input data is not MeDIP-seq/cfMeDIP-seq enrichment data (e.g. whole-genome bisulfite sequencing or array methylation), since MEDIPIPE's QC and quantification models (MEDIPS/QSEA/MEDStrand) assume MeDIP enrichment profiles
- Data is already aligned and QC'd (BAM-level inputs) and only downstream statistical modelling is needed — MEDIPIPE starts from raw FASTQ files
- Single-sample exploratory analysis where the aggregation and group/batch-level QC reporting machinery is unnecessary overhead

## Inputs

- Raw cfMeDIP-seq FASTQ files (single-end or paired-end, with or without spike-in/UMI sequences)
- Sample sequencing data table (per-sample FASTQ paths and metadata)
- ENCODE pre-built genome references (BWA index and annotated regions)
- Single YAML input configuration file (paths, genome references, sample info, experimental settings)

## Outputs

- Aggregated methylation quantification matrices as TAB-delimited bin-sample TXT files (original and filtered, Tabix-indexed)
- MultiQC (v1.0.dev0) aggregated QC report and summarized interactive HTML QC report
- Per-sample QC metrics: FastQC, insertion size metrics, fragmentation size ratios in 1 and 5 Mb windows, MEDIPS saturation/coverage/enrichment QC

## How to apply

Clone the MEDIPIPE repository and build the core environment with Conda/Mamba (e.g. 'mamba env create --file conda_env.yaml'), letting Snakemake automatically deploy per-tool execution environments defined in YAML files. Obtain raw FASTQ files, prepare the sample sequencing data table, and download ENCODE pre-built genome references (BWA index and annotated regions) via the provided shell scripts, then write a single YAML input configuration file specifying working environment paths, genome references, sample information, and experimental settings (single-end/paired-end, spike-in and/or UMI usage). Run the Snakemake Snakefile with --use-conda (e.g. 'snakemake --use-conda --cores N --configfile test/config_template.yaml'), or submit via SLURM using the workflow/sbatch_Snakefile_template.sh pattern so independent per-sample steps run in parallel: Trim Galore (v0.6.7) trimming with FastQC (v0.11.9) QC, UMI-tools (v1.0.1) barcode extraction/dedup when UMIs are present, BWA-MEM (v0.7.17) alignment, SAMtools (v1.17) filtering and duplicate removal, Picard (v2.26.6) CollectInsertSizeMetrics and fragmentation-size ratios in consecutive 1 and 5 Mb windows, and MEDIPS-derived saturation/coverage/enrichment QC. Quantify methylation per sample with three complementary models — MEDIPS (v1.46.0, linear regression), QSEA (v1.20.0, sigmoidal model), and MEDStrand (stranded sigmoid model) — then aggregate QC with MultiQC into an interactive HTML report and aggregate methylation quantifications into TAB-delimited bin-sample TXT files, filtering out sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions, and indexing both original and filtered files with Tabix (v1.17).

## Related tools

- **Snakemake** (Workflow engine that orchestrates all pipeline steps and automatically deploys containerized execution environments) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Environment manager for the core MEDIPIPE environment and per-tool environments)
- **Mamba** (Recommended fast Conda reimplementation for installing the pipeline environment) — https://github.com/mamba-org/mamba
- **SLURM** (Cluster engine supported for parallel per-sample job submission via the sbatch Snakefile template)
- **Trim Galore** (Adapter/quality trimming (v0.6.7) of raw FASTQ reads)
- **FastQC** (Per-sample read QC (v0.11.9) after trimming)
- **UMI-tools** (Barcode extraction and deduplication (v1.0.1) when UMIs are present)
- **BWA-MEM** (Alignment of trimmed reads to the reference genome (v0.7.17))
- **SAMtools** (Alignment filtering and duplicate removal (v1.17))
- **Picard** (CollectInsertSizeMetrics and fragmentation size ratios in consecutive 1 and 5 Mb windows (v2.26.6))
- **MEDIPS** (Methylation quantification with linear regression model (v1.46.0) plus saturation/coverage/enrichment QC)
- **QSEA** (Methylation quantification with sigmoidal model (v1.20.0))
- **MEDStrand** (Stranded sigmoid-model methylation quantification)
- **MultiQC** (Aggregation of per-sample QC into a summarized interactive HTML report (v1.0.dev0))
- **Tabix** (Indexing of original and filtered aggregated methylation TXT files (v1.17))

## Examples

```
snakemake --use-conda --cores 16 --configfile test/config_template.yaml
```

## Evaluation signals

- Aggregated methylation matrices (before and after filtering) exist and are Tabix-indexed for all samples (e.g. all 163 Nassiri et al. 2020 brain cancer samples)
- Sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions are absent from the filtered bin-sample matrices
- MultiQC HTML report contains per-sample FastQC, insertion-size, and fragmentation-window metrics for every sample
- Group/batch-level QC view matches the pattern demonstrated in Supplementary Fig. S2 of the paper
- Methylation quantifications are produced under all three models (MEDIPS linear, QSEA sigmoidal, MEDStrand stranded sigmoid) for each sample

## Limitations

- Designed specifically for (cf)MeDIP-seq enrichment data; not applicable to bisulfite or array methylation data
- Requires a Conda-based Python3 distribution and sufficient cluster or stand-alone compute; per-tool versions are pinned in YAML files, so major version drift may break reproducibility
- Default filtering removes sex chromosomes and the mitochondrial chromosome, which is inappropriate if sex-chromosome methylation is the study focus

## Evidence

- [intro] MEDIPIPE is end-to-end from FASTQ to aggregation: "The MEDIPIPE is designed for automated end-to-end quality control (QC) and analysis of (cf)MeDIP-seq data. The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation"
- [abstract] Portability via Snakemake and Conda: "ease of implementation and reproducibility with Snakemake containerized execution environments that will be automatically deployed via Conda"
- [readme] Flexibility for single/paired-end, spike-in/UMI, and aggregation: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences. It can be applied to individual samples, as well as to aggregate multiple samples from"
- [intro] Cluster engine support: "It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
- [abstract] One-stop solution scope: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
