---
name: eclip-significant-window-filtering
description: Use when you have per-window IP and input read-start counts from tiled
  CLIP/eCLIP windows plus matched input replicates, and need genome-wide enrichment
  p-values to call reproducible RBP binding windows.
when_to_use_negative:
- CLIP datasets lacking matched input samples — Skipper requires them to construct
  the beta-binomial model and cannot analyze such data
- Analyses needing continuous peak calling with flexible fragment-level boundaries
  — Skipper deliberately skips peak calling and uses fixed tiled windows, so it is
  the wrong fit for peak-style outputs
- Windows with fewer total reads than THRESHOLD_MIN across IP and input fractions,
  or skyscraper windows failing GINI_CUTOFF — these are excluded by design rather
  than tested
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Skipper
  role: End-to-end workflow that tiles windows over annotated transcripts, applies
    GINI_CUTOFF and THRESHOLD_MIN filters, and fits GC-stratified beta-binomial models
    for enriched window identification
  repo: https://github.com/YeoLab/skipper
- name: VGAM
  role: R library providing the beta-binomial family used to fit decile-specific overdispersion
    (rho) and mean (mu) parameters
- name: Snakemake
  role: Pipeline manager that orchestrates the enriched window identification stage
    of Skipper (version 9.12.0)
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:0aa3ed5bed9ed99a5643d2afbffc6044a9a1ed8fd10980f278d2ccf0083eec9d
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
  - Skipper
  - Snakemake
  - VGAM
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# eclip-significant-window-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically tests tiled transcriptomic and repetitive-element windows for enrichment of IP read-start signal over matched input in eCLIP data by fitting GC-stratified beta-binomial models and computing per-window p-values, as implemented in Skipper's enriched window identification stage.

## When to use

Use when you have per-window IP and input read-start counts from tiled CLIP/eCLIP windows plus matched input replicates, and need genome-wide enrichment p-values to call reproducible RBP binding windows.

## When NOT to use

- CLIP datasets lacking matched input samples — Skipper requires them to construct the beta-binomial model and cannot analyze such data
- Analyses needing continuous peak calling with flexible fragment-level boundaries — Skipper deliberately skips peak calling and uses fixed tiled windows, so it is the wrong fit for peak-style outputs
- Windows with fewer total reads than THRESHOLD_MIN across IP and input fractions, or skyscraper windows failing GINI_CUTOFF — these are excluded by design rather than tested

## Inputs

- Per-window read-start counts for input and CLIP/IP fractions across tiled transcriptomic windows
- Matched input (size-matched control) BAM files from eCLIP experiments in at least 2 replicates
- Filtered GFF annotation (one best transcript per gene, e.g., from gencode.v49.basic.annotation.gff3)
- Per-window GC content for decile stratification

## Outputs

- Genome-wide table of window coordinates, statistics, and enrichment p-values (all p-values within [0, 1])
- Set of reproducible enriched windows significant in the configured number of CLIP replicates
- Decile-specific beta-binomial parameters (rho and mu) learned from replicate overdispersion

## How to apply

Tile windows over annotated transcripts (GENCODE/Ensembl GFF), drop 'skyscraper' windows flagged by the GINI_CUTOFF filter (default ~0.9; 0.8 stricter, 0.95 more lenient), then stratify surviving windows into GC-content deciles. Within each decile, fit a beta-binomial model (e.g., VGAM beta-binomial family in R) estimating overdispersion rho from replicates, preferring OVERDISPERSION_MODE='input' (multiple input replicates) over 'clip'. Apply the decile-specific rho and mu to compute a per-window IP-vs-input enrichment p-value, excluding windows with fewer than THRESHOLD_MIN total reads across IP and input fractions. Finally combine decile results into one table and mark windows significant in at least the DEFINITION_OF_REPRODUCIBILITY number of CLIP replicates (default 'ALL', minimum 2) as reproducible.

## Related tools

- **Skipper** (End-to-end workflow that tiles windows over annotated transcripts, applies GINI_CUTOFF and THRESHOLD_MIN filters, and fits GC-stratified beta-binomial models for enriched window identification) — https://github.com/YeoLab/skipper
- **VGAM** (R library providing the beta-binomial family used to fit decile-specific overdispersion (rho) and mean (mu) parameters)
- **Snakemake** (Pipeline manager that orchestrates the enriched window identification stage of Skipper (version 9.12.0)) — https://github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file exists and contains a p-value column for every retained window
- All computed p-values lie within [0, 1]
- Number of retained windows is consistent with the THRESHOLD_MIN and GINI_CUTOFF filter settings
- Reproducible windows meet the DEFINITION_OF_REPRODUCIBILITY criterion (default 'ALL' replicates, minimum 2 CLIP replicates)
- Overdispersion rho is estimated within each GC decile (not genome-wide), and estimates derive from input replicates when OVERDISPERSION_MODE='input'

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Fixed windows can miss very narrow or dense binding patterns — GINI_CUTOFF removes 'skyscraper' windows and thresholding drops low-count windows rather than testing them
- Stricter GINI_CUTOFF values (e.g., 0.8) reduce retained windows while lenient values (e.g., 0.95) retain more, so results are sensitive to filter parameterization

## Evidence
