---
name: bin-level-genomic-matrix-generation
description: Use when duplicate-filtered cfMeDIP-seq BAM files must be converted into
  per-bin methylation estimates with correction for CpG density bias, or when relative
  vs. absolute (vs. strand-aware) methylation strategies must be compared per sample.
when_to_use_negative:
- Input data is not enrichment-based cfMeDIP-seq (e.g., whole-genome bisulfite sequencing),
  where CpG-density-bias correction via MeDIP-style sigmoid/linear models is not applicable
- Only raw FASTQ files are available and no alignment/QC has been performed, since
  this module expects the aligned, duplicate-filtered BAM outputs of the upstream
  MEDIPIPE module
- A single consensus methylation track is already provided and the goal is only sample
  aggregation rather than per-bin quantification
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MEDIPS
  role: Linear-regression CpG-density-corrected bin-level relative methylation scoring
    on cfMeDIP-seq BAMs (v1.46.0); also run separately on spike-in alignments
- name: QSEA
  role: Sigmoidal-model CpG-density correction incorporating empirical knowledge to
    estimate absolute bin-level methylation (v1.20.0)
- name: MEDStrand
  role: Stranded sigmoid model inferring strand-aware absolute methylation levels
    per bin (v0.0.0.9000)
- name: Snakemake
  role: Workflow engine that orchestrates the three quantification models and deploys
    their execution environments automatically
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:418e3dc7106c72911bef80096e5d071949f2ba3cf244616f1e6eeb88b1a14b27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - MEDIPS
  - MEDStrand
  - QSEA
  - Snakemake
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bin-level-genomic-matrix-generation

## Summary

Generates bin-sample methylation quantification tables from enrichment-based cfMeDIP-seq alignments using three CpG-density-bias-correcting models in MEDIPIPE's methylation quantification module: MEDIPS (linear regression, relative methylation), QSEA (sigmoidal model with empirical knowledge, absolute methylation), and MEDStrand (stranded sigmoid model, strand-aware absolute methylation).

## When to use

Use when duplicate-filtered cfMeDIP-seq BAM files must be converted into per-bin methylation estimates with correction for CpG density bias, or when relative vs. absolute (vs. strand-aware) methylation strategies must be compared per sample.

## When NOT to use

- Input data is not enrichment-based cfMeDIP-seq (e.g., whole-genome bisulfite sequencing), where CpG-density-bias correction via MeDIP-style sigmoid/linear models is not applicable
- Only raw FASTQ files are available and no alignment/QC has been performed, since this module expects the aligned, duplicate-filtered BAM outputs of the upstream MEDIPIPE module
- A single consensus methylation track is already provided and the goal is only sample aggregation rather than per-bin quantification

## Inputs

- Aligned, duplicate-filtered BAM files per sample (from MEDIPIPE alignment/QC module)
- YAML configuration file specifying single-end vs paired-end reads and spike-in presence
- Spike-in read alignments (when spike-in controls are configured)

## Outputs

- Tab-delimited bin-sample TXT tables of relative methylation scores from MEDIPS (v1.46.0)
- Tab-delimited bin-sample TXT tables of absolute methylation levels from QSEA (v1.20.0)
- Tab-delimited bin-sample TXT tables of strand-aware absolute methylation levels from MEDStrand (v0.0.0.9000)
- Spike-in MEDIPS quantification table (when spike-in controls are declared)

## How to apply

Parse the YAML configuration file to resolve experimental settings (single-end vs paired-end, spike-in presence) and load the aligned, duplicate-filtered BAM files produced by the alignment/QC module. Run MEDIPS (v1.46.0) with its linear regression CpG-density correction to compute bin-level relative methylation scores; run QSEA (v1.20.0) with its sigmoidal model incorporating empirical knowledge to estimate absolute methylation at the same bins; and run MEDStrand (v0.0.0.9000) with its stranded sigmoid model for strand-aware absolute methylation. When spike-in controls are declared in the configuration, run MEDIPS separately on the extracted spike-in read alignments to quantify methylation levels for the controls. Write each model's per-sample quantifications as tab-delimited bin-sample TXT tables ready for the downstream aggregation and filtering module. Choose among the three normalization strategies based on whether relative, absolute, or strand-aware absolute methylation levels are required.

## Related tools

- **MEDIPS** (Linear-regression CpG-density-corrected bin-level relative methylation scoring on cfMeDIP-seq BAMs (v1.46.0); also run separately on spike-in alignments)
- **QSEA** (Sigmoidal-model CpG-density correction incorporating empirical knowledge to estimate absolute bin-level methylation (v1.20.0))
- **MEDStrand** (Stranded sigmoid model inferring strand-aware absolute methylation levels per bin (v0.0.0.9000))
- **Snakemake** (Workflow engine that orchestrates the three quantification models and deploys their execution environments automatically) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- Bin-sample quantification TXT tables are generated for all three models (MEDIPS, QSEA, MEDStrand) for every sample
- MEDStrand requires sufficient read depth for the sigmoid fit — verify the strand-aware table exists and is populated for adequately deep samples
- A spike-in MEDIPS quantification table exists whenever spike-in controls are declared in the YAML configuration
- Tables are tab-delimited with consistent bin coordinates across the three models so they feed the downstream aggregation and filtering module
- Software versions match those pinned in the YAML files (MEDIPS v1.46.0, QSEA v1.20.0, MEDStrand v0.0.0.9000)

## Limitations

- MEDStrand's stranded sigmoid model requires sufficient read depth for the sigmoid fit; shallow samples may fail to produce strand-aware estimates
- The three models target different estimands (relative vs absolute vs strand-aware absolute methylation), so their outputs are not interchangeable without considering the chosen normalization strategy
- Spike-in-based quantification is only available when spike-in controls are present in the sequencing data and declared in the configuration

## Evidence

- [other] Three CpG-density-bias-correcting models: [evidence span withheld — anchor status: not_found]
- [other] Quantification table outputs: [evidence span withheld — anchor status: not_found]
- [readme] End-to-end pipeline scope: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [abstract] One-stop cfMeDIP-seq solution: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
