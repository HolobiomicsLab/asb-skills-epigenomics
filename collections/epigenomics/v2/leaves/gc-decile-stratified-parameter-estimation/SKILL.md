---
name: gc-decile-stratified-parameter-estimation
description: Use when calling RNA-binding sites from CLIP/eCLIP data by testing tiled
  transcriptomic windows for IP-over-input enrichment and you need GC-bias-aware,
  overdispersion-calibrated nulls rather than peak calling.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample — Skipper cannot construct the beta-binomial
  model for statistical testing without these controls
- The goal is de novo peak discovery on unannotated regions rather than testing windows
  over annotated transcripts/repetitive elements
- The annotation GFF has not been cleaned to one best transcript per gene — window
  selection and interpretability degrade
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline whose enriched-window-identification step performs
    the GC-stratified beta-binomial fitting and per-window p-value computation
  repo: github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine (version 9.12.0) used to orchestrate the Skipper pipeline
    and its conda environments
  repo: https://github.com/YeoLab/skipper.git
- name: VGAM
  role: R package supplying the beta-binomial family used to fit overdispersion (rho)
    within each GC decile
- name: STAR
  role: Aligner generating the genome index and the aligned BAMs from which per-window
    read-start counts are derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:3279b731ad77e7d43c3e8d0403d43a7260cea5e1e9e3d436da906aeacdac4b9a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - STAR
  - Skipper
  - Snakemake
  - VGAM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gc-decile-stratified-parameter-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits GC-decile-stratified beta-binomial models to CLIP/IP vs. input read-start counts in tiled transcriptomic windows, estimating overdispersion (rho) separately per GC content bin so that enrichment p-values account for GC bias and count overdispersion in eCLIP data.

## When to use

Use when calling RNA-binding sites from CLIP/eCLIP data by testing tiled transcriptomic windows for IP-over-input enrichment and you need GC-bias-aware, overdispersion-calibrated nulls rather than peak calling.

## When NOT to use

- The CLIP dataset lacks a matched input sample — Skipper cannot construct the beta-binomial model for statistical testing without these controls
- The goal is de novo peak discovery on unannotated regions rather than testing windows over annotated transcripts/repetitive elements
- The annotation GFF has not been cleaned to one best transcript per gene — window selection and interpretability degrade

## Inputs

- Per-window read-start count tables for input samples
- Per-window read-start count tables for CLIP/IP samples
- Tiled transcriptomic window coordinates (from GFF partitioning over GENCODE/Ensembl annotations)
- GC content of each window
- Parameters: GINI_CUTOFF (~0.9), THRESHOLD_MIN, OVERDISPERSION_MODE ('input' recommended), DEFINITION_OF_REPRODUCIBILITY (default 'ALL', minimum 2 CLIP replicates)

## Outputs

- Genome-wide table of window coordinates, statistics, and beta-binomial enrichment p-values
- Reproducibility flag per window (significant in >= DEFINITION_OF_REPRODUCIBILITY CLIP replicates)
- Set of retained/enriched windows with p-values in [0, 1]

## How to apply

Load per-window read-start counts for input and CLIP/IP fractions across windows tiled over annotated transcripts, then drop 'skyscraper' windows flagged by the GINI_CUTOFF filter (default ~0.9; 0.8 is stricter, 0.95 more lenient). Stratify surviving windows into deciles of GC content. Within each decile, fit a beta-binomial model (e.g., VGAM's beta-binomial family in R), estimating overdispersion rho from replicates per the OVERDISPERSION_MODE — estimating from multiple input replicates ('input') is recommended over 'clip'. Apply the decile-specific rho and mu to compute a per-window IP-vs-input enrichment p-value, excluding windows with fewer than THRESHOLD_MIN total reads across the input and IP fractions, then pool decile results into one genome-wide table and mark windows significant in at least DEFINITION_OF_REPRODUCIBILITY (default 'ALL', minimum 2) CLIP replicates as reproducible. The rationale is that GC bias and read-count overdispersion are estimated separately so that p-values are calibrated against windows with comparable sequence composition.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline whose enriched-window-identification step performs the GC-stratified beta-binomial fitting and per-window p-value computation) — github.com/YeoLab/skipper
- **Snakemake** (Workflow engine (version 9.12.0) used to orchestrate the Skipper pipeline and its conda environments) — https://github.com/YeoLab/skipper.git
- **VGAM** (R package supplying the beta-binomial family used to fit overdispersion (rho) within each GC decile)
- **STAR** (Aligner generating the genome index and the aligned BAMs from which per-window read-start counts are derived)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file exists and contains a p-value column with an entry for every tested window
- All reported p-values fall within [0, 1]
- Number of retained windows is consistent with the applied THRESHOLD_MIN and GINI_CUTOFF filters (fewer skyscraper windows survive at stricter GINI_CUTOFF like 0.8)
- Reproducible windows are significant in at least the DEFINITION_OF_REPRODUCIBILITY number of CLIP replicates (default 'ALL', minimum 2)
- GC-decile-specific rho and mu values are estimated from all deciles and applied consistently when pooling results

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- THRESHOLD_MIN excludes low-count windows, so lowly expressed transcripts may be untestable
- Overdispersion estimated from a single replicate (or from 'clip' mode instead of multiple 'input' replicates) is less reliable
- Window selection depends on a properly filtered GFF; overlapping isoforms must be reduced to one best transcript per gene

## Evidence
