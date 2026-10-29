---
name: cfmedip-methylation-quantification
description: Use when estimating bin-level methylation from cfMeDIP-seq enrichment
  data and CpG-density bias must be corrected, e.g. choosing among linear-regression,
  sigmoidal, or strand-aware sigmoid normalization models.
when_to_use_negative:
- Input is raw FASTQ rather than aligned, duplicate-filtered BAM files — the upstream
  alignment/QC module must run first
- Data are not enrichment-based (cf)MeDIP-seq reads, e.g. bisulfite or nanopore methylation
  calls, which need different quantification methods
- MEDStrand is being used but the sample lacks sufficient read depth for the stranded
  sigmoid model fit
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MEDIPS
  role: Linear-regression CpG-density correction producing bin-level relative methylation
    scores; also run on spike-in alignments when spike-ins are configured
- name: QSEA
  role: Sigmoidal model incorporating empirical knowledge to estimate absolute methylation
    levels at the same bins
- name: MEDStrand
  role: Stranded sigmoid model inferring strand-aware absolute methylation levels
- name: Snakemake
  role: Orchestrates the three quantification models with automatically deployed containerized
    execution environments
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:319e2edad9bd73412256c2d2b3dcdd2a8a0397003096bded2b93255de18f0b63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - MEDIPS
  - MEDStrand
  - QSEA
  - Snakemake
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cfmedip-methylation-quantification

## Summary

MEDIPIPE's methylation quantification module estimates relative and absolute methylation levels from enrichment-based cfMeDIP-seq data by offering three CpG-density-bias-correcting normalization models: MEDIPS (linear regression), QSEA (sigmoidal with empirical knowledge), and MEDStrand (stranded sigmoid).

## When to use

Use when estimating bin-level methylation from cfMeDIP-seq enrichment data and CpG-density bias must be corrected, e.g. choosing among linear-regression, sigmoidal, or strand-aware sigmoid normalization models.

## When NOT to use

- Input is raw FASTQ rather than aligned, duplicate-filtered BAM files — the upstream alignment/QC module must run first
- Data are not enrichment-based (cf)MeDIP-seq reads, e.g. bisulfite or nanopore methylation calls, which need different quantification methods
- MEDStrand is being used but the sample lacks sufficient read depth for the stranded sigmoid model fit

## Inputs

- Aligned, duplicate-filtered BAM files per sample from the MEDIPIPE alignment/QC module
- YAML configuration file specifying single-end vs paired-end setting and spike-in presence
- Extracted spike-in read alignment BAMs (when spike-in controls are configured)

## Outputs

- Tab-delimited bin-sample TXT methylation quantification tables for MEDIPS (relative methylation scores)
- Tab-delimited bin-sample TXT tables for QSEA (absolute methylation levels)
- Tab-delimited bin-sample TXT tables for MEDStrand (strand-aware absolute methylation levels)
- MEDIPS quantification table for spike-in controls (when spike-ins are declared)

## How to apply

Parse the YAML configuration file to resolve experimental settings (single-end vs paired-end, spike-in presence) and load the aligned, duplicate-filtered BAM files from the alignment/QC module. Run MEDIPS (v1.46.0) with its linear regression CpG-density correction to compute bin-level relative methylation scores; run QSEA (v1.20.0) with its sigmoidal model incorporating empirical knowledge for absolute methylation levels; and run MEDStrand (v0.0.0.9000) with its stranded sigmoid model for strand-aware absolute methylation. When spike-in controls are declared in the configuration, run MEDIPS separately on the extracted spike-in read alignments to quantify control methylation. Each model's per-sample quantifications are written as tab-delimited bin-sample TXT tables for the downstream aggregation and filtering module. Users choose the normalization strategy appropriate to their data; all three models are orchestrated via Snakemake.

## Related tools

- **MEDIPS** (Linear-regression CpG-density correction producing bin-level relative methylation scores; also run on spike-in alignments when spike-ins are configured)
- **QSEA** (Sigmoidal model incorporating empirical knowledge to estimate absolute methylation levels at the same bins)
- **MEDStrand** (Stranded sigmoid model inferring strand-aware absolute methylation levels)
- **Snakemake** (Orchestrates the three quantification models with automatically deployed containerized execution environments) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE && snakemake --use-conda --cores 8 methylation_quantification
```

## Evaluation signals

- Bin-sample quantification TXT tables are generated for all three models (MEDIPS, QSEA, MEDStrand) for every sample
- The spike-in MEDIPS quantification table exists whenever spike-in controls are declared in the YAML configuration
- MEDStrand completes only on samples with sufficient read depth for the stranded sigmoid fit — flag samples where the fit fails
- Output tables are tab-delimited with a consistent bin-by-sample layout ready for the downstream aggregation and filtering module
- Software versions match the pinned releases (MEDIPS v1.46.0, QSEA v1.20.0, MEDStrand v0.0.0.9000) as specified in the YAML files

## Limitations

- MEDStrand requires sufficient read depth for its sigmoid model to fit; low-depth samples may fail
- The three models produce different quantities (relative vs absolute vs strand-aware absolute), so outputs are not interchangeable without understanding the normalization strategy
- Methylation estimation is confined to CpG-density-corrected bins; accuracy depends on the CpG-density bias models embedded in each tool

## Evidence

- [results] MEDIPIPE's methylation quantification module applies three CpG-density-bias-correcting methods: [evidence span withheld — anchor status: not_found]
- [results] QSEA sigmoidal model: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline scope: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [readme] Snakemake orchestration: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments."
