---
name: spike-in-control-quantification
description: Use when a cfMeDIP-seq pipeline configuration declares spike-in controls
  and methylation levels must be quantified for those controls from extracted spike-in
  alignments.
when_to_use_negative:
- The configuration declares no spike-in controls — the spike-in MEDIPS run and its
  output table should be skipped
- You need absolute or strand-aware methylation estimates for the main samples rather
  than spike-in control quantification — use the QSEA sigmoidal or MEDStrand stranded
  sigmoid runs instead
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: MEDIPS
  role: Run separately on extracted spike-in read alignments, using its linear regression
    CpG-density correction, to quantify methylation levels for the spike-in controls
  repo: https://github.com/pughlab/MEDIPIPE
- name: Snakemake
  role: Orchestrates the spike-in MEDIPS quantification rule within MEDIPIPE, auto-deploying
    the execution environment
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:653643dbd4734bd3677c235601882747714419739acd0dfb5dced4162b58d07e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spike-in-control-quantification

## Summary

Within MEDIPIPE's methylation quantification module, when spike-in controls are declared in the YAML configuration, MEDIPS (v1.46.0) with its linear regression CpG-density correction is run separately on the extracted spike-in read alignments to quantify methylation levels for the controls.

## When to use

Use when a cfMeDIP-seq pipeline configuration declares spike-in controls and methylation levels must be quantified for those controls from extracted spike-in alignments.

## When NOT to use

- The configuration declares no spike-in controls — the spike-in MEDIPS run and its output table should be skipped
- You need absolute or strand-aware methylation estimates for the main samples rather than spike-in control quantification — use the QSEA sigmoidal or MEDStrand stranded sigmoid runs instead

## Inputs

- MEDIPIPE YAML configuration file declaring spike-in controls (and single-end/paired-end, UMI settings)
- Aligned, duplicate-filtered spike-in read alignments (BAM) extracted by the alignment/QC module

## Outputs

- Spike-in MEDIPS quantification table: tab-delimited bin-sample TXT of methylation levels for the controls

## How to apply

First parse the MEDIPIPE YAML configuration file to resolve whether spike-in controls are present (alongside single-end vs paired-end and UMI settings). If spike-in controls are configured, take the aligned, duplicate-filtered spike-in read alignments that were extracted by the alignment/QC module. Run MEDIPS (v1.46.0) separately on these spike-in alignments, applying its linear regression CpG-density correction, to compute methylation levels for the control reads. Write the spike-in MEDIPS quantification as a tab-delimited bin-sample TXT table so it sits alongside the per-sample MEDIPS, QSEA (v1.20.0), and MEDStrand (v0.0.0.9000) quantification tables feeding the downstream aggregation and filtering module. This spike-in track provides an external control signal for normalizing/assessing cfMeDIP-seq methylation quantification across samples.

## Related tools

- **MEDIPS** (Run separately on extracted spike-in read alignments, using its linear regression CpG-density correction, to quantify methylation levels for the spike-in controls) — https://github.com/pughlab/MEDIPIPE
- **Snakemake** (Orchestrates the spike-in MEDIPS quantification rule within MEDIPIPE, auto-deploying the execution environment) — https://github.com/pughlab/MEDIPIPE

## Evaluation signals

- The spike-in MEDIPS quantification table exists as a tab-delimited bin-sample TXT file whenever spike-in controls are declared in the YAML configuration
- The spike-in quantification table is generated for every sample configured with spike-in controls
- The spike-in quantification uses the same bin structure as the per-sample MEDIPS/QSEA/MEDStrand tables so downstream aggregation can consume it
- The spike-in run consumed the extracted spike-in read alignments (duplicate-filtered BAM) rather than the main-sample alignments

## Limitations

- The spike-in quantification is only triggered when spike-in controls are declared in the YAML configuration; without configured controls no spike-in table is produced
- Only MEDIPS (linear regression model) is used for spike-in quantification — the QSEA and MEDStrand absolute/strand-aware models are not applied to spike-in controls in the documented workflow

## Evidence

- [other] MEDIPIPE's methylation quantification module applies three CpG-density-bias-correcting methods for relative and absolute methylation estimation: [evidence span withheld — anchor status: not_found]
- [other] Spike-in MEDIPS run description: [evidence span withheld — anchor status: not_found]
- [other] Spike-in validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline scope and input: "The MEDIPIPE is designed for automated end-to-end quality control (QC) and analysis of (cf)MeDIP-seq data."
- [readme] Flexibility feature mentioning spike-in handling: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
