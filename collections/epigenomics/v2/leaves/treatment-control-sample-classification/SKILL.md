---
name: treatment-control-sample-classification
description: Use when processing compiled FRiP QC records (e.g. id_frip JSON from
  ChiLin's compile_database.sh) and you need treatment-vs-control class proportions
  passing an enrichment cutoff.
when_to_use_negative:
- Input records lack FRiP scores or class labels, so treatment/control stratification
  cannot be performed
- The goal is per-sample pipeline QC reporting on new runs rather than cohort-level
  threshold assessment against the historical atlas
- Samples are peak files without the underlying FRiP JSON QC records produced by ChiLin
  step 7
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ChiLin
  role: Pipeline whose step 7 computes the FRiP scores and whose compile_database.sh
    post-processing script assembles the QC database that is stratified here
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller used by ChiLin to produce the peaks against which FRiP
    scores are calculated
  repo: https://github.com/taoliu/MACS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:981565db2d2d3c8a005973c1ccb11856164da470c359b4a808bf3e897b15cdaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# treatment-control-sample-classification

## Summary

Stratify public ChIP-seq/DNase-seq samples from a compiled ChiLin QC database into ChIP (treatment) versus input (control) classes and evaluate each class against a 1% FRiP threshold to assess ChIP enrichment quality.

## When to use

Use when processing compiled FRiP QC records (e.g. id_frip JSON from ChiLin's compile_database.sh) and you need treatment-vs-control class proportions passing an enrichment cutoff.

## When NOT to use

- Input records lack FRiP scores or class labels, so treatment/control stratification cannot be performed
- The goal is per-sample pipeline QC reporting on new runs rather than cohort-level threshold assessment against the historical atlas
- Samples are peak files without the underlying FRiP JSON QC records produced by ChiLin step 7

## Inputs

- Compiled FRiP QC JSON records (e.g. id_frip) per sample from the ChiLin pipeline
- compile_database.sh post-processing output database over 23,677 public ChIP-seq and DNase-seq samples
- Sample class labels (ChIP/treatment vs input/control)

## Outputs

- frip_cutoff_summary.csv with per-class pass counts and proportions
- Per-class totals (ChIP: 14,866; input: 8,811) and pass proportions (0.809 and 0.486)

## How to apply

Load the compiled QC database built by compile_database.sh from ChiLin FRiP JSON records (e.g. id_frip), covering the 23,677 public ChIP-seq and DNase-seq samples in the quality atlas. Parse the FRiP score from each sample record and stratify samples into ChIP (treatment) versus input (control) classes. Apply a 1% FRiP threshold, counting each sample as passing if its FRiP score is greater than 0.01. Compute pass counts and proportions per class, expecting 12,032/14,866 = 80.9% for ChIP and 4,289/8,811 = 48.6% for input. The rationale is that ChIP samples passing 1% FRiP far more often than inputs supports 1% FRiP as a proper ChIP enrichment reference cutoff.

## Related tools

- **ChiLin** (Pipeline whose step 7 computes the FRiP scores and whose compile_database.sh post-processing script assembles the QC database that is stratified here) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller used by ChiLin to produce the peaks against which FRiP scores are calculated) — https://github.com/taoliu/MACS

## Examples

```
git clone https://github.com/cfce/chilin && bash demo/compile_database.sh
```

## Evaluation signals

- Class totals match the atlas strata: 14,866 ChIP samples and 8,811 input samples
- Pass proportions match reported values within rounding: 0.809 (12,032/14,866) for ChIP and 0.486 (4,289/8,811) for input
- Each sample is scored strictly as FRiP > 0.01 to count as passing
- frip_cutoff_summary.csv contains per-class counts and proportions consistent with the thresholded records
- Sum of per-class totals reconciles against the number of parsed FRiP records in the compiled database

## Limitations

- The 1% FRiP cutoff is validated only against this atlas of 23,677 public samples; other experimental contexts may warrant different thresholds
- ChiLin has no stated software version and the reproducibility claim lacks concrete commands, environment specification, or output artifacts in some sections
- Classification depends on accurate input/control labeling in the compiled database; mislabeled samples would distort the class proportions

## Evidence

- [methods] FRiP scores calculation: "step 7: calculate FRiP scores for each sample."
- [other] compile_database.sh provides the QC database used for stratification: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [full_text] Expected class counts and proportions under the 1% FRiP threshold: [evidence span withheld — anchor status: not_found]
- [full_text] 1% FRiP threshold definition and rationale: [evidence span withheld — anchor status: not_found]
