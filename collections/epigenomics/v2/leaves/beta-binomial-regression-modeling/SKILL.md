---
name: beta-binomial-regression-modeling
description: Use when analyzing tiled transcriptomic (or repetitive-element) windows
  from eCLIP data with matched IP and input read-start counts, and you need GC-bias-aware
  enrichment p-values accounting for count overdispersion.
when_to_use_negative:
- No matched input sample is available — the beta-binomial model cannot be constructed
  and Skipper cannot be applied
- The goal is free-form peak discovery with variable-width peaks rather than fixed-window
  enrichment testing over annotated transcripts
- Read counts come from a single pooled replicate with no input replicates to estimate
  overdispersion rho
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Skipper
  role: End-to-end pipeline implementing the tiled-window beta-binomial enrichment
    test; skips peak calling by tiling windows over annotated transcripts
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine used to orchestrate the Skipper pipeline steps including enriched-window
    identification (version 9.12.0)
  repo: https://github.com/YeoLab/skipper
- name: VGAM
  role: R package providing the beta-binomial family used to fit the decile-specific
    beta-binomial model and estimate overdispersion rho
- name: STAR
  role: Aligner used upstream to generate the genome index and produce the aligned
    reads whose start counts feed the beta-binomial model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:098888841cf2f645e33fa7b2dc4e52fdef1d262f61ae1db5c71224b0e9d0952c
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
  - STAR
  - Skipper
  - Snakemake
  - VGAM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# beta-binomial-regression-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical skill from the Skipper CLIP-seq pipeline: fit a beta-binomial model per GC-content decile to model overdispersion in read-start counts, then compute IP-vs-input enrichment p-values for tiled windows to call RNA-binding sites.

## When to use

Use when analyzing tiled transcriptomic (or repetitive-element) windows from eCLIP data with matched IP and input read-start counts, and you need GC-bias-aware enrichment p-values accounting for count overdispersion.

## When NOT to use

- No matched input sample is available — the beta-binomial model cannot be constructed and Skipper cannot be applied
- The goal is free-form peak discovery with variable-width peaks rather than fixed-window enrichment testing over annotated transcripts
- Read counts come from a single pooled replicate with no input replicates to estimate overdispersion rho

## Inputs

- Per-window read-start counts from IP/CLIP samples (BAM-derived)
- Matched input-control sample read-start counts
- Tiled transcriptomic windows from GFF partitioning (GENCODE/Ensembl annotation)
- Repetitive-element coordinates (e.g., UCSC Table Browser repeats)

## Outputs

- Genome-wide table of window coordinates, statistics, and per-window enrichment p-values
- Set of reproducible enriched windows significant in the required number of CLIP replicates
- Fitted beta-binomial parameters (rho, mu) per GC-content decile

## How to apply

Load per-window read-start counts for input and CLIP/IP fractions across tiled windows, dropping 'skyscraper' windows flagged by the GINI_CUTOFF filter (default ~0.9). Stratify surviving windows into GC-content deciles so GC bias is learned locally rather than globally. Within each decile, fit the beta-binomial model (e.g., via the VGAM beta-binomial family in R), estimating the overdispersion parameter rho from multiple input replicates (OVERDISPERSION_MODE='input' is recommended over 'clip'). Apply the fitted decile-specific rho and mu to compute a per-window enrichment p-value, excluding windows with fewer than THRESHOLD_MIN total reads across input and IP fractions. Combine decile results into a genome-wide table and mark windows significant in at least DEFINITION_OF_REPRODUCIBILITY CLIP replicates (default 'ALL', minimum 2) as reproducible.

## Related tools

- **Skipper** (End-to-end pipeline implementing the tiled-window beta-binomial enrichment test; skips peak calling by tiling windows over annotated transcripts) — https://github.com/YeoLab/skipper
- **Snakemake** (Workflow engine used to orchestrate the Skipper pipeline steps including enriched-window identification (version 9.12.0)) — https://github.com/YeoLab/skipper
- **VGAM** (R package providing the beta-binomial family used to fit the decile-specific beta-binomial model and estimate overdispersion rho)
- **STAR** (Aligner used upstream to generate the genome index and produce the aligned reads whose start counts feed the beta-binomial model)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output file exists with a p-value column for every window
- All computed p-values lie within [0, 1]
- Number of retained windows is consistent with the THRESHOLD_MIN and GINI_CUTOFF filters applied
- Fitted rho/mu differ across GC deciles (GC-stratified fits, not a single global model)
- Reproducible windows satisfy the DEFINITION_OF_REPRODUCIBILITY criterion (default 'ALL' replicates, minimum 2)

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Skyscraper windows with extremely narrow peaks must be removed via GINI_CUTOFF (stricter 0.8, more lenient 0.95) or the model fit degrades
- Windows with total reads below THRESHOLD_MIN across input and IP fractions are excluded, limiting sensitivity on low-abundance transcripts

## Evidence

- [results] Skipper learns overdispersion in read counts and GC bias separately for transcriptomic windows and repetitive elements, then computes p values from the fit beta-binomial distributions: [evidence span withheld — anchor status: not_found]
