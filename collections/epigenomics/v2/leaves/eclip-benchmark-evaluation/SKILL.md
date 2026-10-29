---
name: eclip-benchmark-evaluation
description: Use when candidate eCLIP binding sites must be evaluated against orthogonally
  ascertained true binding sites (e.g., chrM for FASTKD2, splice sites for PRPF8,
  GURUG motif for TARDBP) to compare peak callers.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample, so Skipper cannot construct its beta-binomial
  model and reproducible_enriched_windows cannot be produced for benchmarking
- No orthogonal, RBP-specific truth set exists for the RBP of interest — overlap-based
  precision/recall would be uninterpretable
- The goal is to quantify binding-site count differences rather than benchmark accuracy
  (use the 210%-320% more-sites analysis instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3524
tools:
- name: Skipper
  role: End-to-end workflow producing reproducible_enriched_windows tables from raw
    eCLIP reads for the six benchmarked RBPs
  repo: https://github.com/YeoLab/skipper
- name: CLIPper
  role: Existing peak-calling method whose called-site tables are scored against the
    same truth sets for comparison
  repo: https://github.com/YeoLab/clipper
- name: Snakemake
  role: Workflow engine (version 9.12.0) executing the Skipper pipeline used to generate
    the benchmark inputs
- name: STAR
  role: Aligner used within Skipper to map eCLIP reads before window calling
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin (version 1.4.0) enabling HPC execution of the Skipper
    Snakemake pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:bf47f64347f373f972d6a3ec1ecf624181703965443e47fc730f12e4cd427d16
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
  - CLIPper
  - SLURM
  - STAR
  - Skipper
  - Snakemake
  - conda
  - snakemake-executor-plugin-slurm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# eclip-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark CLIP-seq analysis methods (Skipper vs. CLIPper, Piranha, etc.) by scoring called binding sites against RBP-specific orthogonal truth sets and computing precision, relative recall, and F1 per method-RBP pair across six example eCLIPs.

## When to use

Use when candidate eCLIP binding sites must be evaluated against orthogonally ascertained true binding sites (e.g., chrM for FASTKD2, splice sites for PRPF8, GURUG motif for TARDBP) to compare peak callers.

## When NOT to use

- The CLIP dataset lacks a matched input sample, so Skipper cannot construct its beta-binomial model and reproducible_enriched_windows cannot be produced for benchmarking
- No orthogonal, RBP-specific truth set exists for the RBP of interest — overlap-based precision/recall would be uninterpretable
- The goal is to quantify binding-site count differences rather than benchmark accuracy (use the 210%-320% more-sites analysis instead)

## Inputs

- reproducible_enriched_windows tables from Skipper runs on six RBP eCLIP datasets (ENCODE single-end protocol, ≥2 replicates)
- called-site/peak tables from comparison methods (CLIPper, Piranha) on the same eCLIP datasets
- RBP-specific orthogonal truth sets (chrM transcripts, PUM2 affinity scores, 5' splice site annotations, GURUG motif sites, Y RNA genes, histone stem-loop motifs)

## Outputs

- table_s3_method_benchmark_prf.csv: precision, relative recall, and F1 for every method-RBP combination across the six RBPs (equivalent to Figure 2G / Table S3)
- Per-site TP/FP/FN labels from truth-set overlap for each method-RBP pair

## How to apply

Run Skipper via Snakemake 9.12.0 (SLURM executor plugin 1.4.0, STAR alignment, ENCODE4 single-end protocol, at least 2 CLIP replicates per sample) on the eCLIP datasets for six RBPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP) to obtain reproducible_enriched_windows tables, and gather called-site tables from comparison methods such as CLIPper and Piranha. For each RBP, define an orthogonal positive truth set: chrM binding for FASTKD2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP, Y RNA genes and chrM for TROVE2, and histone stem-loop motifs for SLBP. Mark each called site as TP, FP, or FN by overlap with the RBP-specific truth set, then compute precision TP/(TP+FP), recall TP/(TP+FN), and F1 (harmonic mean) per method-RBP pair plus aggregates. Rationale: truth sets are orthogonal to CLIP signal, so overlaps directly assess whether a method recovers biologically validated binding rather than merely abundant peaks.

## Related tools

- **Skipper** (End-to-end workflow producing reproducible_enriched_windows tables from raw eCLIP reads for the six benchmarked RBPs) — https://github.com/YeoLab/skipper
- **CLIPper** (Existing peak-calling method whose called-site tables are scored against the same truth sets for comparison) — https://github.com/YeoLab/clipper
- **Snakemake** (Workflow engine (version 9.12.0) executing the Skipper pipeline used to generate the benchmark inputs)
- **STAR** (Aligner used within Skipper to map eCLIP reads before window calling)
- **snakemake-executor-plugin-slurm** (SLURM executor plugin (version 1.4.0) enabling HPC execution of the Skipper Snakemake pipeline)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Output table contains precision, recall, and F1 entries for every method-RBP combination across all six RBPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP)
- All metric values fall in the range [0, 1]
- Each metric is computed from an explicit TP/FP/FN count derived from truth-set overlaps, not from arbitrary site rankings
- Skipper attains the highest F1 score across all six example eCLIPs relative to CLIPper and Piranha
- Exceptions to the >10% F1-improvement pattern are limited to CLIPper output for FASTKD2 and PUM2

## Limitations

- CLIPper output for FASTKD2 and PUM2 were the only cases where Skipper's F1 improvement did not exceed other methods' by more than 10%, so the benchmark does not show uniform dominance
- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed or benchmarked this way
- Truth sets are RBP-specific and orthogonal ascertainment is only available for a limited set of RBPs, constraining which eCLIPs can enter the benchmark

## Evidence

- [results] Skipper attained the highest F1 score across all example eCLIPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP); CLIPper output for FASTKD2 and PUM2 were the only cases where Skipper's improvement did not exceed other methods' F1 score by more than 10%.: [evidence span withheld — anchor status: not_found]
- [supplementary] Define, for each RBP, an orthogonal positive truth set: chrM binding for FASTKD2 and TROVE2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP.: [evidence span withheld — anchor status: not_found]
