---
name: overdispersion-estimation-from-replicates
description: Use when calling enriched CLIP/eCLIP windows from tiled transcriptomic
  or repetitive-element windows, where per-window read-start counts in IP and matched
  input are available and counts are overdispersed and GC-biased.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample — Skipper requires one to construct
  the beta-binomial model for statistical testing
- Read counts per window are below THRESHOLD_MIN, so the beta-binomial test cannot
  be reliably computed for that window
- The goal is peak-level calling with variable-width intervals rather than fixed tiled-window
  enrichment testing
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: End-to-end eCLIP workflow implementing the GC-stratified beta-binomial enrichment
    test over tiled windows
  repo: github.com/YeoLab/skipper
- name: VGAM
  role: R package whose beta-binomial family is used to fit the per-GC-decile beta-binomial
    model and estimate overdispersion rho
- name: Snakemake
  role: Workflow engine that executes the Skipper pipeline stages, including enriched
    window identification
  repo: github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:b964d6606c790406c5c4fe13c7743b0db6dba87ba48dec7dd58bffc9038c16a5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - Skipper
  - Snakemake
  - VGAM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Overdispersion estimation from replicates

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skipper's statistical core: it estimates read-count overdispersion (rho) and GC-stratified mean bias (mu) from input/CLIP replicates using a beta-binomial model, then computes per-window IP-vs-input enrichment p-values to call binding sites.

## When to use

Use when calling enriched CLIP/eCLIP windows from tiled transcriptomic or repetitive-element windows, where per-window read-start counts in IP and matched input are available and counts are overdispersed and GC-biased.

## When NOT to use

- The CLIP dataset lacks a matched input sample — Skipper requires one to construct the beta-binomial model for statistical testing
- Read counts per window are below THRESHOLD_MIN, so the beta-binomial test cannot be reliably computed for that window
- The goal is peak-level calling with variable-width intervals rather than fixed tiled-window enrichment testing

## Inputs

- Per-window read-start count table for input and CLIP/IP fractions across tiled transcriptomic windows
- Per-window counts for repetitive-element windows
- GC content per window (for decile stratification)
- Window-level Gini coefficients (for GINI_CUTOFF skyscraper filtering)
- Configuration parameters: OVERDISPERSION_MODE, GINI_CUTOFF, THRESHOLD_MIN, DEFINITION_OF_REPRODUCIBILITY

## Outputs

- Genome-wide table of window coordinates, statistics, and per-window enrichment p-values (all within [0, 1])
- Reproducibility calls marking windows significant in at least the required number of CLIP replicates

## How to apply

Load per-window read-start counts for input and CLIP/IP fractions across tiled windows and drop 'skyscraper' windows flagged by GINI_CUTOFF (default ~0.9; stricter 0.8, lenient 0.95). Stratify surviving windows into deciles by GC content, since GC bias must be learned separately for transcriptomic windows and repetitive elements. Within each GC decile, fit the beta-binomial model (e.g., via the VGAM beta-binomial family in R), estimating overdispersion rho from replicates according to OVERDISPERSION_MODE — estimating from multiple input replicates ('input') is recommended over 'clip'. Apply the fitted decile-specific rho and mu to compute a per-window IP-vs-input enrichment p-value, excluding windows with fewer than THRESHOLD_MIN total reads across input and IP, then combine decile results into a genome-wide table and mark windows reproducible if significant in at least DEFINITION_OF_REPRODUCIBILITY CLIP replicates (default 'ALL', minimum 2 replicates).

## Related tools

- **Skipper** (End-to-end eCLIP workflow implementing the GC-stratified beta-binomial enrichment test over tiled windows) — github.com/YeoLab/skipper
- **VGAM** (R package whose beta-binomial family is used to fit the per-GC-decile beta-binomial model and estimate overdispersion rho)
- **Snakemake** (Workflow engine that executes the Skipper pipeline stages, including enriched window identification) — github.com/YeoLab/skipper

## Examples

```
conda create -n snakemake9 snakemake=9.12.0; snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file exists and contains a p-value column for every window
- All computed p-values lie within [0, 1]
- Number of retained windows is consistent with the applied THRESHOLD_MIN and GINI_CUTOFF filters
- Reproducible windows are significant in at least DEFINITION_OF_REPRODUCIBILITY CLIP replicates (default 'ALL', minimum 2)
- Overdispersion rho and mu are estimated per GC-content decile (e.g., 10 deciles) for both transcriptomic windows and repetitive elements

## Limitations

- Skipper requires a matched input sample; CLIP datasets lacking these controls cannot be analyzed
- Windows with fewer than THRESHOLD_MIN total reads across input and IP are excluded from testing
- 'Skyscraper' windows with very narrow peaks are removed by GINI_CUTOFF, so ultra-focused signal cannot be called in transcriptomic windows

## Evidence

- [methods] GC-decile beta-binomial fit: [evidence span withheld — anchor status: not_found]
- [discussion] Matched-input requirement: [evidence span withheld — anchor status: not_found]
