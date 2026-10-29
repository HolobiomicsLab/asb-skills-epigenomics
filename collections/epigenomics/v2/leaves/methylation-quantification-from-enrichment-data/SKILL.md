---
name: methylation-quantification-from-enrichment-data
description: Use when you have cfMeDIP-seq (or MeDIP-seq) aligned enrichment data
  from raw FASTQs and need per-sample methylation quantification plus aggregated,
  filtered bin-by-sample methylation matrices across a cohort.
when_to_use_negative:
- Input is already a methylation matrix or beta-value table — quantification from
  enrichment data assumes raw FASTQ/aligned reads as the starting point
- Data type is not MeDIP-seq/cfMeDIP-seq enrichment data (e.g. whole-genome bisulfite
  sequencing or methylation arrays), since MEDIPS/QSEA/MEDStrand models assume immunoprecipitation-based
  enrichment
- Analysis goal is only per-read QC without methylation quantification or cohort-level
  aggregation
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: MEDIPS
  role: Methylation quantification per sample using a linear regression model (v1.46.0)
  repo: https://github.com/pughlab/MEDIPIPE
- name: QSEA
  role: Methylation quantification per sample using a sigmoidal model (v1.20.0)
  repo: https://github.com/pughlab/MEDIPIPE
- name: MEDStrand
  role: Stranded methylation quantification using a sigmoid model (v0.0.0.9000)
  repo: https://github.com/pughlab/MEDIPIPE
- name: Tabix
  role: Indexes both original and filtered aggregated bin-sample TXT files (v1.17)
- name: Snakemake
  role: Orchestrates the quantification and aggregation workflow with automatically
    deployed Conda execution environments
  repo: https://github.com/pughlab/MEDIPIPE
- name: SAMtools
  role: Post-alignment filtering and duplicate removal feeding into quantification
    (v1.17)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:e8b979470e29db055a4a69a9395f613b5963f6eb4a4a09aa51cae48eda3d204c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - BWA-MEM
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
yes_rate: 0.8571428571428571
n_claims: 14
---

# methylation-quantification-from-enrichment-data

## Summary

Quantify methylation from cfMeDIP-seq enrichment data and aggregate it across samples, using three complementary models — MEDIPS (linear regression), QSEA (sigmoidal model), and MEDStrand (stranded sigmoid model) — then produce Tabix-indexed bin-sample matrices with blacklist and sex-chromosome filtering as implemented in MEDIPIPE.

## When to use

Use when you have cfMeDIP-seq (or MeDIP-seq) aligned enrichment data from raw FASTQs and need per-sample methylation quantification plus aggregated, filtered bin-by-sample methylation matrices across a cohort.

## When NOT to use

- Input is already a methylation matrix or beta-value table — quantification from enrichment data assumes raw FASTQ/aligned reads as the starting point
- Data type is not MeDIP-seq/cfMeDIP-seq enrichment data (e.g. whole-genome bisulfite sequencing or methylation arrays), since MEDIPS/QSEA/MEDStrand models assume immunoprecipitation-based enrichment
- Analysis goal is only per-read QC without methylation quantification or cohort-level aggregation

## Inputs

- Raw cfMeDIP-seq FASTQ files (single-end or paired-end, with or without spike-in/UMI sequences)
- Sample sequencing data table (sample sheet)
- ENCODE pre-built genome references (BWA index and annotated regions)
- Single YAML input configuration file (working environment paths, genome references, sample information, experimental settings)

## Outputs

- Per-sample methylation quantifications from MEDIPS (linear regression), QSEA (sigmoidal), and MEDStrand (stranded sigmoid)
- Aggregated TAB-delimited bin-sample TXT methylation matrices (original and filtered)
- Filtered matrices excluding sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions
- Tabix (v1.17) indexes for both original and filtered aggregated files

## How to apply

After alignment with BWA-MEM (v0.7.17) and post-alignment filtering/deduplication with SAMtools (v1.17), quantify methylation per sample with three models: MEDIPS v1.46.0 using a linear regression, QSEA v1.20.0 using a sigmoidal model, and MEDStrand v0.0.0.9000 using a stranded sigmoid model. Aggregate per-sample quantifications into TAB-delimited bin-sample TXT files, then filter out sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions to produce a cleaned matrix. Index both the original and the filtered aggregated files with Tabix (v1.17) so downstream region queries and differential analyses can access bins efficiently. Pipeline orchestration via Snakemake with --use-conda automatically deploys the per-tool Conda execution environments, and the whole workflow is configured through a single YAML input file specifying working environment paths, genome references, sample information, and experimental settings (single-end/paired-end, spike-in and/or UMI usage).

## Related tools

- **MEDIPS** (Methylation quantification per sample using a linear regression model (v1.46.0)) — https://github.com/pughlab/MEDIPIPE
- **QSEA** (Methylation quantification per sample using a sigmoidal model (v1.20.0)) — https://github.com/pughlab/MEDIPIPE
- **MEDStrand** (Stranded methylation quantification using a sigmoid model (v0.0.0.9000)) — https://github.com/pughlab/MEDIPIPE
- **Tabix** (Indexes both original and filtered aggregated bin-sample TXT files (v1.17))
- **Snakemake** (Orchestrates the quantification and aggregation workflow with automatically deployed Conda execution environments) — https://github.com/pughlab/MEDIPIPE
- **SAMtools** (Post-alignment filtering and duplicate removal feeding into quantification (v1.17))

## Examples

```
snakemake --use-conda --cores 8 --configfile test/config_template.yaml
```

## Evaluation signals

- Aggregated methylation matrices (before and after filtering) are produced as TAB-delimited bin-sample TXT files for every sample in the cohort (e.g. all 163 Nassiri et al. 2020 samples)
- Filtered matrices contain no bins from sex chromosomes, the mitochondrial chromosome, or ENCODE blacklist regions
- Both original and filtered aggregated files have valid Tabix indexes (.tbi) and are queryable
- Quantification outputs exist for all three models — MEDIPS, QSEA, and MEDStrand — for each sample
- Aggregated outputs reproduce the group/batch-level QC view demonstrated in Supplementary Fig. S2 of the article

## Limitations

- Quantification models differ: MEDIPS uses linear regression while QSEA uses a sigmoidal model and MEDStrand a stranded sigmoid model, so results are not interchangeable and model choice is a decision point
- Filtering removes sex chromosomes and the mitochondrial chromosome, so methylation on chrM and sex chromosomes is not retained in the final filtered matrix
- The pipeline is designed specifically for (cf)MeDIP-seq enrichment data and assumes raw FASTQ inputs with configurable single-end/paired-end, spike-in, and UMI settings via the YAML configuration

## Evidence

- [abstract] MEDIPIPE provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [intro] MEDIPIPE handles single-end or paired-end reads with or without spike-in/UMI sequences, for individual samples or large-scale aggregation: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences. It can be applied to individual samples, as well as to aggregate multiple samples from"
- [intro] The pipeline runs from raw FASTQ files through QC, alignment, methylation quantification and aggregation: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation"
- [readme] Snakemake automatically deploys execution environments across cluster engines or stand-alone machines: [evidence span withheld — anchor status: not_found]
- [intro] MEDIPIPE supports different cluster engines such as SLURM or stand-alone machines: "It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
