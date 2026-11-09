---
name: cpg-density-bias-correction
description: Use when quantifying methylation from cfMeDIP-seq (MeDIP-seq) enrichment
  data — e.g. duplicate-filtered BAM files after alignment/QC — where CpG density
  bias must be corrected, and a choice among relative, absolute, or strand-aware methylation
  estimates is needed.
when_to_use_negative:
- Input is bisulfite or enzymatic methyl-seq (WGBS/EM-seq) data, where methylation
  is called directly from conversion rather than enrichment and no CpG-density-bias
  correction model is needed
- BAM files have not yet passed the upstream alignment and duplicate-filtering QC
  steps of the pipeline
- Read depth is insufficient for MEDStrand's sigmoid fit, leaving only MEDIPS and
  QSEA as viable normalization options
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MEDIPS
  role: CpG-density-bias-correcting linear regression model used for bin-level relative
    methylation estimation and, when spike-ins are present, control methylation quantification
    (v1.46.0)
- name: QSEA
  role: Sigmoidal normalization model incorporating empirical knowledge, used to estimate
    absolute methylation levels at bins (v1.20.0)
- name: MEDStrand
  role: Stranded sigmoid model used to infer strand-aware absolute methylation levels
    (v0.0.0.9000)
- name: Snakemake
  role: Workflow engine that orchestrates the three methylation quantification models
    within MEDIPIPE's end-to-end cfMeDIP-seq pipeline
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:b71aa7bfc1b29d9bda7dce884b8d923b5523e610bb3d4db6519a539b3b8a0818
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cpg-density-bias-correction

## Summary

This skill reconstructs MEDIPIPE's methylation quantification module, which corrects CpG-density bias in enrichment-based cfMeDIP-seq data using three alternative normalization models — MEDIPS (linear regression, relative methylation), QSEA (sigmoidal model with empirical knowledge, absolute methylation), and MEDStrand (stranded sigmoid model, strand-aware absolute methylation).

## When to use

Use when quantifying methylation from cfMeDIP-seq (MeDIP-seq) enrichment data — e.g. duplicate-filtered BAM files after alignment/QC — where CpG density bias must be corrected, and a choice among relative, absolute, or strand-aware methylation estimates is needed.

## When NOT to use

- Input is bisulfite or enzymatic methyl-seq (WGBS/EM-seq) data, where methylation is called directly from conversion rather than enrichment and no CpG-density-bias correction model is needed
- BAM files have not yet passed the upstream alignment and duplicate-filtering QC steps of the pipeline
- Read depth is insufficient for MEDStrand's sigmoid fit, leaving only MEDIPS and QSEA as viable normalization options

## Inputs

- Aligned, duplicate-filtered BAM files from the alignment/QC module (single-end or paired-end cfMeDIP-seq reads, optionally with spike-in)
- YAML configuration file resolving experimental settings (single-end vs paired-end, spike-in presence)
- Extracted spike-in read alignments (when spike-in controls are configured)

## Outputs

- MEDIPS bin-sample relative methylation table (tab-delimited TXT)
- QSEA bin-sample absolute methylation table (tab-delimited TXT)
- MEDStrand strand-aware absolute methylation table (tab-delimited TXT)
- Spike-in MEDIPS quantification table (when spike-in controls are declared)

## How to apply

Parse the pipeline's YAML configuration file to resolve experimental settings (single-end vs paired-end reads, presence of spike-in controls) and locate the aligned, duplicate-filtered BAM files produced by the upstream alignment/QC module. Run MEDIPS (v1.46.0) with its linear regression CpG-density correction to compute bin-level relative methylation scores per sample, and in parallel run QSEA (v1.20.0), whose sigmoidal model incorporates empirical knowledge to estimate absolute methylation at the same bins. Run MEDStrand (v0.0.0.9000), which applies a stranded sigmoid model to infer strand-aware absolute methylation, allowing the user to choose the appropriate normalization strategy for the downstream aggregation and filtering module. When spike-in controls are declared in the configuration, run MEDIPS separately on the extracted spike-in read alignments to quantify control methylation levels. All three models write per-sample quantifications as tab-delimited bin-sample TXT tables that feed the aggregation stage.

## Related tools

- **MEDIPS** (CpG-density-bias-correcting linear regression model used for bin-level relative methylation estimation and, when spike-ins are present, control methylation quantification (v1.46.0))
- **QSEA** (Sigmoidal normalization model incorporating empirical knowledge, used to estimate absolute methylation levels at bins (v1.20.0))
- **MEDStrand** (Stranded sigmoid model used to infer strand-aware absolute methylation levels (v0.0.0.9000))
- **Snakemake** (Workflow engine that orchestrates the three methylation quantification models within MEDIPIPE's end-to-end cfMeDIP-seq pipeline) — https://github.com/pughlab/MEDIPIPE

## Examples

```
snakemake --cores 8 --use-conda --configfile config.yaml methylation_quantification
```

## Evaluation signals

- Bin-sample quantification tables (tab-delimited TXT) are generated for all three models — MEDIPS, QSEA, and MEDStrand — for every sample
- MEDStrand produces a valid output only when read depth is sufficient for the sigmoid fit; a missing MEDStrand table on deep data indicates a failure
- A spike-in MEDIPS quantification table exists whenever spike-in controls are declared in the YAML configuration
- Downstream aggregation and filtering module receives all expected bin-sample TXT tables without missing-sample errors

## Limitations

- MEDStrand requires sufficient read depth for the stranded sigmoid model to fit; at low depth its strand-aware absolute methylation estimates are unreliable
- MEDIPS, QSEA, and MEDStrand yield different methylation scales (relative vs absolute vs strand-aware absolute), so the user must choose the appropriate normalization strategy rather than naively pooling outputs
- Spike-in-based methylation quantification via MEDIPS is only run when spike-in controls are declared in the configuration

## Evidence

- [methods] MEDIPIPE's methylation quantification module applies three CpG-density-bias-correcting methods for relative and absolute methylation estimation: MEDIPS (v1.46.0) using a linear regression model, QSEA (v1.20.0) using a sigmoidal model: [evidence span withheld — anchor status: not_found]
- [abstract] MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [readme] MEDIPIPE was developed with [Snakemake], which will automatically deploy the execution environments. Users can also specify the version of softwares to be installed in YAML files: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments. Users can also specify the version of"
- [intro] The MEDIPIPE is designed for automated end-to-end quality control (QC) and analysis of (cf)MeDIP-seq data. The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation: "The MEDIPIPE is designed for automated end-to-end quality control (QC) and analysis of (cf)MeDIP-seq data. The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation"
