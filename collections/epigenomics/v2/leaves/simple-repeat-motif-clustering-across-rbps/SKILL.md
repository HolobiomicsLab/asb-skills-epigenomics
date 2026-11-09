---
name: simple-repeat-motif-clustering-across-rbps
description: Use when analyzing Skipper's enriched repeat element results across many
  eCLIP experiments and you need to characterize RBP sequence preferences for simple
  repeats, or to validate reproduction of the reported 216/219 enriched-experiment
  recovery.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample, since Skipper requires a matched
  input to construct the beta-binomial model for statistical testing
- The goal is transcript-level binding site annotation rather than repeat/multi-mapping
  sequence binding, where the standard window-based GFF workflow is sufficient
- The organism or genome build has no RepeatMasker annotation available from the UCSC
  Table Browser to construct REPEAT_BED
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3524
tools:
- name: Skipper
  role: End-to-end workflow that tests annotated repeat elements for enrichment using
    a beta-binomial framework with pseudocount-based normalization and input-driven
    overdispersion
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine that runs Skipper, aggregating reads per RepeatMasker element
    alongside the window-level analysis
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns eCLIP IP and matched input reads to the genome prior to repeat enrichment
    testing
- name: SLURM
  role: HPC executor plugin used to parallelize Skipper jobs across cluster compute
    nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:f425276aab28b4c5ccee7b9bb5fdcd5572b44ef24f0e0ebae9352fe6f0a8a742
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper
  tools:
  - SLURM
  - STAR
  - Skipper
  - Snakemake
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# simple-repeat-motif-clustering-across-rbps

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters eCLIP experiments by the classes of repetitive elements Skipper identifies as significantly enriched, exposing RBP binding preferences for simple repeats (e.g., GU-rich elements dominating, QKI recovering (UAC)n and RBFOX2 recovering UGUG-containing motifs). It operationalizes Skipper's RepeatMasker-based beta-binomial testing output across 219 ENCODE eCLIP datasets.

## When to use

Use when analyzing Skipper's enriched repeat element results across many eCLIP experiments and you need to characterize RBP sequence preferences for simple repeats, or to validate reproduction of the reported 216/219 enriched-experiment recovery.

## When NOT to use

- The CLIP dataset lacks a matched input sample, since Skipper requires a matched input to construct the beta-binomial model for statistical testing
- The goal is transcript-level binding site annotation rather than repeat/multi-mapping sequence binding, where the standard window-based GFF workflow is sufficient
- The organism or genome build has no RepeatMasker annotation available from the UCSC Table Browser to construct REPEAT_BED

## Inputs

- gzipped, sorted, non-overlapping RepeatMasker repeat annotation BED (chr,start,end,label,score,strand,name,class,family,proportion_gc) generated via UCSC Table Browser
- STAR-aligned eCLIP IP BAM files and matched input replicates from ENCODE
- Skipper REPEAT_TABLE configuration referencing the repeat annotation
- Per-element enrichment test results from Skipper's beta-binomial framework

## Outputs

- repeat_enrichment_summary.tsv reporting the fraction of enhanced CLIP experiments with at least one significantly enriched repeat element
- Simple-repeat class clustering of experiments (GU-rich dominance, QKI-(UAC)n, RBFOX2-UGUG motif recovery)
- Strand- and age-stratified LINE1 enrichment tables for MATR3 and SAFB2

## How to apply

First run the Skipper Snakemake workflow on STAR-aligned ENCODE eCLIP IP and matched input replicates with REPEAT_TABLE and REPEAT_BED inputs so reads aggregate per RepeatMasker element, testing each element with the beta-binomial framework using NORMALIZATION_MODE='new', OVERDISPERSION_MODE='input', GINI_CUTOFF=0.9, THRESHOLD_MIN as configured, and DEFINITION_OF_REPRODUCIBILITY='ALL'. Tabulate the fraction of enhanced CLIP experiments with at least one significantly enriched repeat element and compare against the reported 99% (216/219). Then cluster experiments by their enriched simple-repeat classes and verify the reported pattern that GU-rich elements dominate, that QKI recovers (UAC)n and RBFOX2 recovers UGUG-containing motifs. For MATR3 and SAFB2, further stratify enriched LINE1 elements by strand and element age to confirm the reported strand/age patterns. Deviations beyond these reported values indicate a failed reproduction.

## Related tools

- **Skipper** (End-to-end workflow that tests annotated repeat elements for enrichment using a beta-binomial framework with pseudocount-based normalization and input-driven overdispersion) — https://github.com/YeoLab/skipper
- **Snakemake** (Workflow engine that runs Skipper, aggregating reads per RepeatMasker element alongside the window-level analysis) — https://github.com/YeoLab/skipper
- **STAR** (Aligns eCLIP IP and matched input reads to the genome prior to repeat enrichment testing)
- **SLURM** (HPC executor plugin used to parallelize Skipper jobs across cluster compute nodes)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The summary table must reproduce the reported statistic that bound repeat elements are identified for 99% of enhanced CLIP experiments (216 of 219)
- The simple-repeat clustering must recover GU-rich element dominance across experiments
- QKI should recover (UAC)n-containing enriched motifs and RBFOX2 should recover UGUG-containing motifs
- MATR3 and SAFB2 stratified LINE1 enrichments must show the reported strand and element age patterns
- Deviations beyond these reported values indicate a failed reproduction

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls currently cannot be analyzed using Skipper
- Few CLIP-seq analysis tools attempt to test for binding to multi-mapping sequences and repetitive elements, so cross-tool validation options are limited
- Repeat coordinates must be supplied as a properly prepared (gzipped, sorted, non-overlapping) RepeatMasker BED file; mis-formed REPEAT_BED inputs will undermine per-element testing

## Evidence
