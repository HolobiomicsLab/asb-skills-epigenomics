---
name: multi-sample-qc-report-aggregation
description: Use when per-sample QC and methylation outputs (FastQC, SAMtools, Picard,
  MEDIPS/QSEA/MEDStrand quantifications) must be pooled across many samples, groups,
  or batches into a summarized report and region-queryable aggregated matrices.
when_to_use_negative:
- Input is still raw FASTQ or unaligned BAMs — upstream QC (Trim Galore, FastQC),
  alignment (BWA-MEM), and per-sample methylation quantification must be complete
  before aggregation
- Only a single sample is being processed without cohort-level comparison, so aggregated
  bin-sample matrices and group/batch QC views provide no added value
- Quantifications come from a non-(cf)MeDIP-seq assay whose bin coordinates or QC
  report formats are incompatible with the MEDIPIPE aggregation rules
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Snakemake
  role: Orchestrates the aggregation module rules with automatically deployed Conda
    execution environments
  repo: https://github.com/pughlab/MEDIPIPE
- name: MultiQC
  role: Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into
    one summarized interactive HTML report (v1.0.dev0, with embedded script)
- name: Tabix
  role: Indexes the sorted, bgzip-compressed original and filtered bin-sample matrices
    (v1.17) for rapid region queries
- name: MEDIPS
  role: Source of linear-regression methylation quantifications aggregated into bin-sample
    matrices (v1.46.0)
- name: QSEA
  role: Source of sigmoidal-model methylation quantifications aggregated into bin-sample
    matrices (v1.20.0)
- name: MEDStrand
  role: Source of stranded sigmoid-model methylation quantifications aggregated into
    bin-sample matrices (v0.0.0.9000)
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:1f4ed59c930ec065883a4ec9a50d2982f47e7bde62be444413739ebcaae6a94c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# multi-sample-qc-report-aggregation

## Summary

Aggregate per-sample QC metrics and methylation quantifications from a (cf)MeDIP-seq pipeline (e.g. MEDIPIPE) into MultiQC-derived interactive HTML QC reports and TAB-delimited bin-sample TXT matrices, filtered and Tabix-indexed so large sample cohorts spanning multiple groups or batches can be inspected jointly.

## When to use

Use when per-sample QC and methylation outputs (FastQC, SAMtools, Picard, MEDIPS/QSEA/MEDStrand quantifications) must be pooled across many samples, groups, or batches into a summarized report and region-queryable aggregated matrices.

## When NOT to use

- Input is still raw FASTQ or unaligned BAMs — upstream QC (Trim Galore, FastQC), alignment (BWA-MEM), and per-sample methylation quantification must be complete before aggregation
- Only a single sample is being processed without cohort-level comparison, so aggregated bin-sample matrices and group/batch QC views provide no added value
- Quantifications come from a non-(cf)MeDIP-seq assay whose bin coordinates or QC report formats are incompatible with the MEDIPIPE aggregation rules

## Inputs

- Per-sample QC report files from FastQC (v0.11.9), SAMtools (v1.17), Picard CollectInsertSizeMetrics, and MEDIPS saturation/coverage/enrichment QC
- Per-sample methylation quantification files from MEDIPS (v1.46.0), QSEA (v1.20.0), and MEDStrand (v0.0.0.9000)
- ENCODE blacklist region annotations (Amemiya et al. 2019) and genome reference annotations
- Snakemake YAML configuration (configfile e.g. test/config_template.yaml) specifying sample information, genome references, and experimental settings

## Outputs

- Summarized interactive HTML QC report (MultiQC v1.0.dev0) with selected QC metrics for all aggregated samples across groups/batches
- TAB-delimited bin-sample TXT methylation matrices (rows = genomic bins, columns = samples), original and filtered versions
- Sorted, bgzip-compressed matrix files with Tabix (v1.17) .tbi indexes

## How to apply

Run the Snakemake aggregation rules after per-sample QC, alignment (BWA-MEM), and methylation quantification complete: first aggregate per-sample QC reports from FASTQC, SAMtools, Picard, and MEDIPS with MultiQC (v1.0.dev0) plus an embedded script, generating a summarized HTML report of selected QC metrics for interactive comparison across groups or batches. Then aggregate each methylation quantification type (MEDIPS v1.46.0 linear regression, QSEA v1.20.0 sigmoidal model, MEDStrand v0.0.0.9000 stranded sigmoid model) into TAB-delimited bin-sample TXT matrices with genomic bins as rows and samples as columns. Uniformly filter each matrix by removing bins on sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions (Amemiya et al. 2019), keeping both the original and filtered versions. Sort, bgzip-compress both versions, and index with Tabix (v1.17) so genomic regions of interest can be rapidly retrieved.

## Related tools

- **Snakemake** (Orchestrates the aggregation module rules with automatically deployed Conda execution environments) — https://github.com/pughlab/MEDIPIPE
- **MultiQC** (Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into one summarized interactive HTML report (v1.0.dev0, with embedded script))
- **Tabix** (Indexes the sorted, bgzip-compressed original and filtered bin-sample matrices (v1.17) for rapid region queries)
- **MEDIPS** (Source of linear-regression methylation quantifications aggregated into bin-sample matrices (v1.46.0))
- **QSEA** (Source of sigmoidal-model methylation quantifications aggregated into bin-sample matrices (v1.20.0))
- **MEDStrand** (Source of stranded sigmoid-model methylation quantifications aggregated into bin-sample matrices (v0.0.0.9000))

## Examples

```
snakemake --use-conda --cores 16 --configfile test/config_template.yaml
```

## Evaluation signals

- Both the original and the filtered aggregated quantification files exist for each quantification type (MEDIPS, QSEA, MEDStrand)
- The aggregated matrices are bgzip-compressed, Tabix-indexed (.tbi present), and a test region query returns the expected bins
- The filtered matrices contain no bins from sex chromosomes, the mitochondrial chromosome, or ENCODE blacklist regions (Amemiya et al. 2019)
- The HTML QC report renders the selected QC metrics for all aggregated samples (e.g. the 163-sample Nassiri et al. 2020 demonstration set spanning 6 brain cancer subtypes)

## Limitations

- Filtering is fixed to removing sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions; the pipeline keeps both original and filtered versions but does not expose custom exclusion sets
- Aggregation assumes quantifications were produced by the pipeline's supported tools (MEDIPS, QSEA, MEDStrand) and QC from FASTQC/SAMtools/Picard/MEDIPS; other formats require adaptation
- Applies to (cf)MeDIP-seq data with single-end or paired-end reads, with or without spike-in/UMI sequences; per-tool execution environments must be deployed via Conda (--use-conda)

## Evidence

- [abstract] Abstract: one-stop solution: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [intro] Intro: aggregation scope: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
- [full_text] Task card: filtering and indexing: [evidence span withheld — anchor status: not_found]
- [full_text] Task card: MultiQC HTML report: [evidence span withheld — anchor status: not_found]
- [full_text] Task card: Tabix indexing: [evidence span withheld — anchor status: not_found]
- [readme] README: pipeline scope: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
