---
name: methylation-matrix-aggregation-across-samples
description: Use when a completed per-sample (cf)MeDIP-seq methylation quantification
  run must be combined across many samples, groups, or batches into genome-bin x sample
  matrices with standardized region filtering and region-queryable output.
when_to_use_negative:
- Only a single sample is being analyzed and no cross-sample aggregation is required
- Input is raw FASTQ data needing QC, alignment, and methylation quantification first
  — this skill is the final downstream stage
- Data are not (cf)MeDIP-seq methylation quantifications, e.g. WGBS or array methylation
  data
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Snakemake
  role: Orchestrates the aggregation, filtering, compression, and QC-report steps
    with automatically deployed containerized execution environments
  repo: https://github.com/pughlab/MEDIPIPE
- name: MultiQC
  role: Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into
    a summarized HTML report (v1.0.dev0)
  repo: https://github.com/pughlab/MEDIPIPE
- name: Tabix
  role: Indexes the sorted, bgzip-compressed bin-sample matrices (v1.17) for rapid
    retrieval of genomic regions of interest
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:26c48f5ad6a3ab7e5933fb956db736b40d702f9748a7ae69ff8e2ce4a7ead16a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - MultiQC
  - Snakemake
  - Tabix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# methylation-matrix-aggregation-across-samples

## Summary

Aggregates per-sample cfMeDIP-seq methylation quantifications (from MEDIPS, QSEA, MEDStrand) into TAB-delimited bin-sample TXT matrices, applies uniform region filtering (sex chromosomes, chrM, ENCODE blacklist), and Tabix-indexes the results alongside a MultiQC HTML report.

## When to use

Use when a completed per-sample (cf)MeDIP-seq methylation quantification run must be combined across many samples, groups, or batches into genome-bin x sample matrices with standardized region filtering and region-queryable output.

## When NOT to use

- Only a single sample is being analyzed and no cross-sample aggregation is required
- Input is raw FASTQ data needing QC, alignment, and methylation quantification first — this skill is the final downstream stage
- Data are not (cf)MeDIP-seq methylation quantifications, e.g. WGBS or array methylation data

## Inputs

- per-sample methylation quantification files from MEDIPS, QSEA, and MEDStrand
- per-sample QC reports from FASTQC, SAMtools, Picard, and MEDIPS
- ENCODE blacklist regions (Amemiya et al. 2019) annotation
- sample metadata defining groups/batches

## Outputs

- TAB-delimited bin-sample TXT matrices per quantification type (rows = genomic bins, columns = samples), original and filtered versions
- sorted, bgzip-compressed matrix files with Tabix (v1.17) indexes
- MultiQC-based summarized HTML QC report with selected QC metrics across all aggregated samples

## How to apply

Collect the per-sample methylation quantification files produced by MEDIPS, QSEA, and MEDStrand and aggregate each quantification type into TAB-delimited bin-sample TXT matrices with genomic bins as rows and samples as columns. Uniformly filter each matrix by removing bins on sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions (Amemiya et al. 2019), retaining both the original and filtered versions. Sort and bgzip-compress both matrix versions and index them with Tabix (v1.17) so genomic regions of interest can be rapidly retrieved. Aggregate the per-sample QC reports (FASTQC, SAMtools, Picard, MEDIPS) with MultiQC (v1.0.dev0) plus an embedded script, and generate a summarized HTML QC report so users can interactively compare sample QC metrics across groups or batches (e.g. the 163-sample, 6-subtype brain cancer demonstration set).

## Related tools

- **Snakemake** (Orchestrates the aggregation, filtering, compression, and QC-report steps with automatically deployed containerized execution environments) — https://github.com/pughlab/MEDIPIPE
- **MultiQC** (Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into a summarized HTML report (v1.0.dev0)) — https://github.com/pughlab/MEDIPIPE
- **Tabix** (Indexes the sorted, bgzip-compressed bin-sample matrices (v1.17) for rapid retrieval of genomic regions of interest) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- Both the original and the filtered aggregated quantification files exist for each quantification type (MEDIPS, QSEA, MEDStrand)
- Matrix files are sorted, bgzip-compressed, Tabix-indexed, and region-queryable (a test region query returns expected bins)
- Filtered matrices contain no bins from sex chromosomes, the mitochondrial chromosome, or ENCODE blacklist regions
- Matrix dimensions match the expected bins x aggregated samples (e.g. 163 samples across 6 brain cancer subtypes)
- The HTML QC report renders the selected QC metrics for every aggregated sample

## Limitations

- Filters are fixed to sex chromosomes, chrM, and ENCODE blacklist regions; custom exclusion regions are not described
- Aggregation assumes per-sample quantifications were produced upstream by MEDIPS, QSEA, or MEDStrand from (cf)MeDIP-seq data; other assay types are out of scope
- Software versions are pinned (Tabix v1.17, MultiQC v1.0.dev0) and executed via Snakemake/Conda environments, so behavior depends on the deployed environment resolving correctly

## Evidence

- [abstract] MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [readme] pipeline scope from FASTQ through aggregation: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [readme] aggregation of multiple samples in large-scale profiling: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
