---
name: precision-recall-f1-computation
description: Use when comparing called binding sites from CLIP-seq methods against
  independently ascertained truth sets (e.g., chrM binding, splice sites, known motifs)
  to quantify which method recovers true RBP targets .
when_to_use_negative:
- No orthogonal truth set is available for the RBP of interest — the TP/FP/FN classification
  requires an independently ascertained positive set (e.g., splice sites, motifs,
  chrM binding).
- The goal is to count total binding sites rather than assess accuracy — Skipper's
  higher site counts (210%-320% more transcriptomic sites) are a separate result from
  the F1 benchmark.
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline producing reproducible_enriched_windows tables whose
    sites are scored against truth sets
  repo: https://github.com/YeoLab/skipper
- name: CLIPper
  role: Existing peak-calling method whose output is benchmarked against the same
    truth sets
  repo: https://github.com/YeoLab/clipper
- name: Snakemake
  role: Workflow manager (v9.12.0) used to execute the Skipper pipeline that generates
    the calls being scored
- name: STAR
  role: Aligner used within Skipper to align eCLIP reads before window calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:9c7ec438c7dc93e710d6e6c7793fd5620ca4f3e25813732c4939697af2acb460
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# precision-recall-f1-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks CLIP-seq peak-calling methods (Skipper, CLIPper, Piranha) by scoring called binding sites against RBP-specific orthogonal truth sets and computing precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 per method-RBP pair.

## When to use

Use when comparing called binding sites from CLIP-seq methods against independently ascertained truth sets (e.g., chrM binding, splice sites, known motifs) to quantify which method recovers true RBP targets best.

## When NOT to use

- No orthogonal truth set is available for the RBP of interest — the TP/FP/FN classification requires an independently ascertained positive set (e.g., splice sites, motifs, chrM binding).
- The goal is to count total binding sites rather than assess accuracy — Skipper's higher site counts (210%-320% more transcriptomic sites) are a separate result from the F1 benchmark.

## Inputs

- reproducible_enriched_windows tables from Skipper runs on eCLIP datasets for the six RBPs
- called-site/peak tables from existing peak-calling methods (e.g., CLIPper, Piranha)
- RBP-specific orthogonal truth sets: chrM binding (FASTKD2, TROVE2), PUM2 affinity-score sites, 5' splice sites (PRPF8), GURUG motif sites (TARDBP), Y RNA genes (TROVE2), histone stem-loop motifs (SLBP)

## Outputs

- Per method-RBP table of precision, recall, and F1 values (equivalent to Figure 2G / Table S3), written to table_s3_method_benchmark_prf.csv
- Explicit TP/FP/FN counts derived from truth-set overlaps for every method-RBP combination

## How to apply

For each RBP (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP), obtain called-site tables from each method: run Skipper via Snakemake 9.12.0 (SLURM executor plugin 1.4.0, STAR alignment, ENCODE4 single-end protocol, >=2 CLIP replicates per sample) to produce reproducible_enriched_windows tables, and gather peak calls from existing methods such as CLIPper and Piranha. Define an RBP-specific orthogonal positive truth set per target: chrM binding for FASTKD2 and TROVE2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP, Y RNA genes for TROVE2, and histone stem-loop motifs for SLBP. Mark each called binding site as true positive, false positive, or false negative by overlap with the truth set, then compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 as the harmonic mean of precision and recall for every method-RBP pair, plus aggregates across the six RBPs. In the Skipper benchmark, Skipper attained the highest F1 across all six example eCLIPs, with CLIPper output for FASTKD2 and PUM2 the only cases where Skipper's improvement did not exceed other methods' F1 by more than 10%.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline producing reproducible_enriched_windows tables whose sites are scored against truth sets) — https://github.com/YeoLab/skipper
- **CLIPper** (Existing peak-calling method whose output is benchmarked against the same truth sets) — https://github.com/YeoLab/clipper
- **Snakemake** (Workflow manager (v9.12.0) used to execute the Skipper pipeline that generates the calls being scored)
- **STAR** (Aligner used within Skipper to align eCLIP reads before window calling)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm  # then compute P/R/F1 from reproducible_enriched_windows vs truth-set overlaps into table_s3_method_benchmark_prf.csv
```

## Evaluation signals

- Output table contains precision, recall, and F1 entries for every method-RBP combination across all six RBPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP).
- All metric values fall in the range [0, 1].
- Each metric is computed from an explicit TP/FP/FN count derived from documented truth-set overlaps, not from implicit scoring.
- F1 equals the harmonic mean of the reported precision and recall for each method-RBP pair.
- Aggregate F1 across the six RBPs should rank Skipper highest, with FASTKD2 and PUM2 (vs. CLIPper) the only pairs where Skipper's margin over other methods is not >10%.

## Limitations

- Truth sets are proxy-based and RBP-specific (e.g., chrM binding for FASTKD2/TROVE2, GURUG motif for TARDBP); sites outside these proxies are counted as false positives or false negatives by construction.
- F1 improvement depends on the quality of the orthogonal truth set; conclusions may not generalize to RBPs lacking such independent evidence.
- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed and thus cannot enter the benchmark.

## Evidence

- [results] Skipper attained the highest F1 score across all example eCLIPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP): [evidence span withheld — anchor status: not_found]
- [results] CLIPper margin exception: [evidence span withheld — anchor status: not_found]
- [supplementary] truth set definition: [evidence span withheld — anchor status: not_found]
- [supplementary] metric definitions: [evidence span withheld — anchor status: not_found]
