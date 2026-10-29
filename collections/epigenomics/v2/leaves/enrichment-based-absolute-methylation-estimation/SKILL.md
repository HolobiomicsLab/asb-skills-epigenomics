---
name: enrichment-based-absolute-methylation-estimation
description: Use when estimating methylation levels from cfMeDIP/(cf)MeDIP-seq enrichment
  data where CpG density bias must be corrected, and a choice among linear, sigmoidal,
  or strand-aware normalization models is needed.
when_to_use_negative:
- Input is raw FASTQ that has not yet passed through MEDIPIPE's QC and alignment module
  producing duplicate-filtered BAMs
- Data are not enrichment-based MeDIP-seq (e.g. whole-genome bisulfite sequencing),
  where CpG-density-bias correction of MeDIP signals is not required
tools:
- name: MEDIPS
  role: Linear-regression CpG-density correction for bin-level relative methylation
    scores; also used on spike-in alignments
- name: QSEA
  role: Sigmoidal model incorporating empirical knowledge for absolute methylation
    estimation at bins
- name: MEDStrand
  role: Stranded sigmoid model inferring strand-aware absolute methylation levels
- name: Snakemake
  role: Orchestrates the methylation quantification module steps with containerized
    Conda environments; software versions pinned in YAML files
  repo: https://snakemake.readthedocs.io/en/stable/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:4ec5f405654e5f1510dddc36e105ecdf82e1aa4a409aaa341b26f9bc625923f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MEDIPS
  - MEDStrand
  - QSEA
  - Snakemake
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# enrichment-based-absolute-methylation-estimation

## Summary

Estimate CpG-density-corrected relative and absolute methylation levels from enrichment-based cfMeDIP-seq data using MEDIPIPE's three alternative quantification models: MEDIPS (v1.46.0, linear regression), QSEA (v1.20.0, sigmoidal with empirical knowledge), and MEDStrand (v0.0.0.9000, stranded sigmoid model).

## When to use

Use when estimating methylation levels from cfMeDIP/(cf)MeDIP-seq enrichment data where CpG density bias must be corrected, and a choice among linear, sigmoidal, or strand-aware normalization models is needed.

## When NOT to use

- Input is raw FASTQ that has not yet passed through MEDIPIPE's QC and alignment module producing duplicate-filtered BAMs
- Data are not enrichment-based MeDIP-seq (e.g. whole-genome bisulfite sequencing), where CpG-density-bias correction of MeDIP signals is not required

## Inputs

- Aligned, duplicate-filtered BAM files from the MEDIPIPE alignment/QC module
- YAML configuration file (single-end vs paired-end, spike-in presence/absence)
- Spike-in read alignments (when spike-in controls are configured)

## Outputs

- Tab-delimited bin-sample TXT quantification tables for MEDIPS (relative methylation scores)
- Tab-delimited bin-sample TXT quantification tables for QSEA (absolute methylation levels)
- Tab-delimited bin-sample TXT quantification tables for MEDStrand (strand-aware absolute methylation levels)
- Spike-in MEDIPS quantification table (when spike-in controls are declared)

## How to apply

Parse MEDIPIPE's YAML configuration file to resolve experimental settings (single-end vs paired-end, spike-in presence) and load the aligned, duplicate-filtered BAM files produced by the alignment/QC module. Run MEDIPS (v1.46.0) with its linear regression CpG-density correction to compute bin-level relative methylation scores per sample, then run QSEA (v1.20.0) with its sigmoidal model incorporating empirical knowledge to estimate absolute methylation levels at the same bins, and MEDStrand (v0.0.0.9000) with its stranded sigmoid model for strand-aware absolute methylation. When spike-in controls are declared in the configuration, run MEDIPS separately on the extracted spike-in read alignments to quantify methylation for the controls. Users choose the normalization strategy appropriate to their data among the three CpG-density-bias-correcting methods.

## Related tools

- **MEDIPS** (Linear-regression CpG-density correction for bin-level relative methylation scores; also used on spike-in alignments)
- **QSEA** (Sigmoidal model incorporating empirical knowledge for absolute methylation estimation at bins)
- **MEDStrand** (Stranded sigmoid model inferring strand-aware absolute methylation levels)
- **Snakemake** (Orchestrates the methylation quantification module steps with containerized Conda environments; software versions pinned in YAML files) — https://snakemake.readthedocs.io/en/stable/index.html

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- Bin-sample quantification TXT tables are generated for all three models (MEDIPS, QSEA, MEDStrand) for every sample
- MEDStrand quantification succeeds only when sufficient read depth is available for the sigmoid fit; absence on deep-enough data signals failure
- A spike-in MEDIPS quantification table exists whenever spike-in controls are declared in the configuration
- Tables are tab-delimited, bin-by-sample formatted, and ready for the downstream aggregation and filtering module
- Software versions match those pinned in the YAML files (MEDIPS v1.46.0, QSEA v1.20.0, MEDStrand v0.0.0.9000)

## Limitations

- MEDStrand requires sufficient read depth for its sigmoid fit; strand-aware absolute methylation cannot be quantified otherwise
- Spike-in-based MEDIPS quantification is only performed when spike-in controls are declared in the YAML configuration
- The three normalization models (linear, sigmoidal, stranded sigmoid) give different relative vs absolute methylation estimates; users must choose the appropriate strategy

## Evidence

- [methods] three-model quantification finding: [evidence span withheld — anchor status: not_found]
- [readme] README pipeline scope: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [intro] intro end-to-end scope: "The MEDIPIPE is designed for automated end-to-end quality control (QC) and analysis of (cf)MeDIP-seq data."
