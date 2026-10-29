---
name: orthogonal-binding-site-truth-set-ascertainment
description: Use when benchmarking a CLIP-seq binding-site caller (e.g., Skipper,
  CLIPper, Piranha) and an independent, biology-grounded positive set exists for an
  RBP (e.g., chrM binding for FASTKD2/TROVE2, 5' splice sites for PRPF8, GURUG motif
  for TARDBP, histone stem-loops for SLBP).
when_to_use_negative:
- The CLIP dataset lacks a matched input sample or the RBP has no orthogonal biological
  truth set (e.g., no known motif, no subcellular compartment, no splice-site signature),
  so TP/FP/FN cannot be defined independently of the callers being compared
- The goal is calling new binding sites, not benchmarking existing callers — this
  skill scores methods against fixed truth sets and does not itself produce binding-site
  calls
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: Skipper
  role: Produces the reproducible_enriched_windows binding-site calls being benchmarked
    against the truth sets; run end-to-end via Snakemake with STAR alignment
  repo: https://github.com/YeoLab/skipper
- name: CLIPper
  role: Existing peak-calling method whose called-site tables are scored against the
    same truth sets as Skipper
  repo: https://github.com/YeoLab/clipper
- name: Snakemake
  role: Workflow manager (version 9.12.0) used to execute the Skipper pipeline that
    generates the evaluated binding-site tables
- name: STAR
  role: Aligner used within Skipper to align eCLIP reads (ENCODE4 single-end protocol)
    prior to enriched-window calling
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin (version 1.4.0) enabling parallel Skipper execution
    on HPC clusters for the six eCLIP runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:d5fc7321348cf8093495de9a02d600b9aba9a6badeb14282153c2dc2f395e7aa
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
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# orthogonal-binding-site-truth-set-ascertainment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that evaluates CLIP-seq peak-calling methods (Skipper vs CLIPper, Piranha) against RBP-specific, orthogonally ascertained truth sets rather than method-to-method agreement, scoring each caller by precision, recall, and F1 over six example eCLIPs.

## When to use

Use when benchmarking a CLIP-seq binding-site caller (e.g., Skipper, CLIPper, Piranha) and an independent, biology-grounded positive set exists for an RBP (e.g., chrM binding for FASTKD2/TROVE2, 5' splice sites for PRPF8, GURUG motif for TARDBP, histone stem-loops for SLBP).

## When NOT to use

- The CLIP dataset lacks a matched input sample or the RBP has no orthogonal biological truth set (e.g., no known motif, no subcellular compartment, no splice-site signature), so TP/FP/FN cannot be defined independently of the callers being compared
- The goal is calling new binding sites, not benchmarking existing callers — this skill scores methods against fixed truth sets and does not itself produce binding-site calls

## Inputs

- reproducible_enriched_windows tables produced by Skipper for FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, and SLBP eCLIPs (ENCODE data, >=2 replicates, ENCODE4 single-end STAR-aligned)
- Called-site tables from comparison peak-calling methods (e.g., CLIPper, Piranha) on the same eCLIP datasets
- Orthogonal truth sets: chrM transcripts, PUM2 RNA affinity-score-derived sites, 5' splice sites, GURUG motif sites, Y RNA gene coordinates, histone stem-loop motif coordinates

## Outputs

- Per-method-RBP precision, recall, and F1 values with explicit TP/FP/FN counts
- table_s3_method_benchmark_prf.csv: a method benchmark table equivalent to Figure 2G / Table S3 with all metric values in [0, 1] for every method-RBP combination

## How to apply

Run Skipper (Snakemake 9.12.0 with the snakemake-executor-plugin-slurm 1.4.0 executor, STAR alignment under the ENCODE4 single-end protocol, at least 2 CLIP replicates per sample) on eCLIP datasets for the six RBPs to obtain reproducible_enriched_windows tables, and gather called-site tables from the comparison methods. For each RBP, define an RBP-specific truth set: chrM binding for FASTKD2 and TROVE2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP, Y RNA genes for TROVE2, and histone stem-loop motifs for SLBP. Mark every called binding site as TP, FP, or FN by overlap with its RBP truth set, then compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 per method-RBP pair plus aggregates across the six RBPs, assembling them into a Table S3-equivalent CSV. The rationale is that agreement between callers does not establish correctness; overlap with independent biological truth does.

## Related tools

- **Skipper** (Produces the reproducible_enriched_windows binding-site calls being benchmarked against the truth sets; run end-to-end via Snakemake with STAR alignment) — https://github.com/YeoLab/skipper
- **CLIPper** (Existing peak-calling method whose called-site tables are scored against the same truth sets as Skipper) — https://github.com/YeoLab/clipper
- **Snakemake** (Workflow manager (version 9.12.0) used to execute the Skipper pipeline that generates the evaluated binding-site tables)
- **STAR** (Aligner used within Skipper to align eCLIP reads (ENCODE4 single-end protocol) prior to enriched-window calling)
- **snakemake-executor-plugin-slurm** (SLURM executor plugin (version 1.4.0) enabling parallel Skipper execution on HPC clusters for the six eCLIP runs)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The output table contains precision, recall, and F1 entries for every method-RBP combination across all six RBPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP)
- All metric values fall in the range [0, 1], and each is computed from an explicit TP/FP/FN count derived from truth-set overlaps, not from method-to-method comparison
- Skipper attains the highest F1 score across all six example eCLIPs
- CLIPper output for FASTKD2 and PUM2 are the only cases where Skipper's improvement does not exceed the other methods' F1 score by more than 10%
- The assembled table is equivalent in structure to Figure 2G / Table S3 of the publication

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed, limiting which experiments can be benchmarked
- The benchmark is only as good as the orthogonally ascertained truth sets; for RBPs without a compartment, motif, or functional signature (e.g., splice sites or chrM binding), no independent TP definition exists
- Truth sets such as chrM binding, splice sites, and known motifs cover only a subset of the transcriptome, so precision/recall are estimated on those restricted regions, not transcriptome-wide

## Evidence

- [results] Skipper attained the highest F1 across all example eCLIPs: [evidence span withheld — anchor status: not_found]
