---
name: tabix-indexed-genomic-matrix-generation
description: Use when per-sample cfMeDIP-seq methylation quantification files (e.g.
  MEDIPS, QSEA, or MEDStrand outputs) must be aggregated into bin-by-sample matrices
  and made region-queryable across many samples or batches.
when_to_use_negative:
- Input is raw FASTQ data that has not yet undergone MEDIPIPE's QC, alignment, and
  per-sample methylation quantification steps
- Only a single sample is being analyzed and no cross-sample aggregation or region-queryable
  matrix is needed
- Analysis requires read-level or peak-level cfMeDIP-seq data rather than binned,
  aggregated methylation summaries
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: MEDIPS
  role: Produces per-sample linear-regression methylation quantifications that feed
    the bin-sample matrix aggregation
- name: QSEA
  role: Produces per-sample sigmoidal-model methylation quantifications that feed
    the bin-sample matrix aggregation
- name: MEDStrand
  role: Produces per-sample stranded sigmoid-model methylation quantifications that
    feed the bin-sample matrix aggregation
- name: Tabix
  role: Indexes the bgzip-compressed original and filtered aggregated matrices (v1.17)
    for rapid genomic-region retrieval
- name: MultiQC
  role: Aggregates per-sample QC reports from FASTQC, SAMtools, Picard, and MEDIPS
    (v1.0.dev0) into a summarized interactive HTML report
- name: Snakemake
  role: Orchestrates the aggregation and filtering rules with automatically deployed
    Conda execution environments
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:deb05803ca07a8f244f63f904281a1a89ae93d57da5fddb10f6d264c024f2291
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
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# tabix-indexed-genomic-matrix-generation

## Summary

MEDIPIPE's aggregation module: per-sample methylation quantifications (MEDIPS, QSEA, MEDStrand) are combined into TAB-delimited bin-sample TXT matrices, uniformly filtered to remove sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions, then sorted, bgzip-compressed, and indexed with Tabix (v1.17) for rapid region queries.

## When to use

Use when per-sample cfMeDIP-seq methylation quantification files (e.g. MEDIPS, QSEA, or MEDStrand outputs) must be aggregated into bin-by-sample matrices and made region-queryable across many samples or batches.

## When NOT to use

- Input is raw FASTQ data that has not yet undergone MEDIPIPE's QC, alignment, and per-sample methylation quantification steps
- Only a single sample is being analyzed and no cross-sample aggregation or region-queryable matrix is needed
- Analysis requires read-level or peak-level cfMeDIP-seq data rather than binned, aggregated methylation summaries

## Inputs

- Per-sample methylation quantification files from MEDIPS (v1.46.0, linear regression)
- Per-sample methylation quantification files from QSEA (v1.20.0, sigmoidal model)
- Per-sample methylation quantification files from MEDStrand (stranded sigmoid model)
- ENCODE blacklist regions (Amemiya et al. 2019) annotation
- Per-sample QC reports from FASTQC, SAMtools, Picard, and MEDIPS (for MultiQC aggregation)

## Outputs

- Original (unfiltered) TAB-delimited bin-sample TXT methylation matrix (rows = genomic bins, columns = samples)
- Filtered TAB-delimited bin-sample TXT methylation matrix (sex chromosomes, chrM, and ENCODE blacklist regions removed)
- bgzip-compressed, sorted, Tabix-indexed (v1.17) versions of both matrices
- MultiQC (v1.0.dev0) aggregated QC report
- Summarized interactive HTML QC report showing selected metrics per sample across groups/batches

## How to apply

Collect the per-sample methylation quantification files produced by MEDIPS (v1.46.0, linear regression), QSEA (v1.20.0, sigmoidal model), and MEDStrand (stranded sigmoid model) and aggregate each quantification type into TAB-delimited bin-sample TXT matrices with genomic bins as rows and samples as columns. Uniformly filter each aggregated matrix by removing bins on sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions (Amemiya et al. 2019), retaining both the original and the filtered versions. Sort and bgzip-compress both versions and index them with Tabix (v1.17) so genomic regions of interest can be rapidly retrieved. In parallel, aggregate per-sample QC reports (FASTQC, SAMtools, Picard, MEDIPS) with MultiQC (v1.0.dev0) via the embedded script and generate a summarized interactive HTML QC report. In MEDIPIPE these steps are executed by the Snakemake pipeline so they run reproducibly after the upstream per-sample QC, alignment, and quantification rules complete.

## Related tools

- **MEDIPS** (Produces per-sample linear-regression methylation quantifications that feed the bin-sample matrix aggregation)
- **QSEA** (Produces per-sample sigmoidal-model methylation quantifications that feed the bin-sample matrix aggregation)
- **MEDStrand** (Produces per-sample stranded sigmoid-model methylation quantifications that feed the bin-sample matrix aggregation)
- **Tabix** (Indexes the bgzip-compressed original and filtered aggregated matrices (v1.17) for rapid genomic-region retrieval)
- **MultiQC** (Aggregates per-sample QC reports from FASTQC, SAMtools, Picard, and MEDIPS (v1.0.dev0) into a summarized interactive HTML report)
- **Snakemake** (Orchestrates the aggregation and filtering rules with automatically deployed Conda execution environments) — https://github.com/pughlab/MEDIPIPE

## Examples

```
snakemake --use-conda --cores 8 --configfile test/config_template.yaml
```

## Evaluation signals

- Both the original and filtered aggregated quantification files exist and are Tabix-indexed (.tbi present) and region-queryable
- Filtered matrices contain no bins from sex chromosomes, the mitochondrial chromosome, or ENCODE blacklist regions (Amemiya et al. 2019)
- Aggregated matrices have genomic bins as rows and all expected samples as columns (e.g. the 163-sample Nassiri et al. 2020 demonstration set spanning 6 brain cancer subtypes)
- Files are sorted and bgzip-compressed before indexing so tabix region queries succeed
- The summarized HTML QC report renders the selected QC metrics for all aggregated samples across groups/batches

## Limitations

- Filtering is applied uniformly (sex chromosomes, chrM, ENCODE blacklist regions); practitioners needing different region masks must handle them downstream
- Tabix indexing requires the matrices to be sorted and bgzip-compressed; unsorted output will fail indexing
- The aggregation module depends on successful completion of upstream per-sample QC, alignment, and MEDIPS/QSEA/MEDStrand quantification rules in the Snakemake workflow

## Evidence

- [other] Filtering and Tabix indexing of aggregated matrices: [evidence span withheld — anchor status: not_found]
- [other] MultiQC aggregation step: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline scope: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [readme] Aggregation applicability: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
