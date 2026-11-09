---
name: motif-sequence-truth-matching
description: Use when comparing called-site outputs from CLIP-seq peak callers (Skipper,
  CLIPper, Piranha) on eCLIP data and an independent motif-, sequence-, or feature-derived
  positive truth set is available for the assayed RBP.
when_to_use_negative:
- No orthogonally derived truth set (motif, splice site, chrM, structural RNA, or
  affinity-score data) exists for the RBP being evaluated — overlap-based TP/FP/FN
  assignment cannot be performed
- The goal is peak calling itself rather than benchmarking caller performance; use
  Skipper's pipeline (GFF partitioning, enriched window identification, fine-mapping)
  instead
- CLIP datasets lack matched input samples required for Skipper's beta-binomial statistical
  model, so Skipper output — one side of the comparison — cannot be generated
tools:
- name: Skipper
  role: End-to-end pipeline run via Snakemake to produce reproducible_enriched_windows
    tables for the six RBPs being benchmarked
  repo: https://github.com/YeoLab/skipper
- name: CLIPper
  role: Existing peak-calling method whose called-site output (BED) is one of the
    methods evaluated against the truth sets
  repo: https://github.com/YeoLab/clipper
- name: Piranha
  role: Existing peak-calling method whose called-site output is benchmarked against
    the truth sets alongside Skipper and CLIPper
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper runs that generate the benchmarked
    windows
- name: STAR
  role: Aligner generating the alignments used by Skipper prior to window calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:1a9b451748e294559ac60dcee6373bde304f09ad7d1bec984cc46f3e80d8db7c
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
  - Piranha
  - SLURM
  - STAR
  - Skipper
  - Snakemake 9.12.0
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# motif-sequence-truth-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking CLIP-seq called binding sites (from Skipper, CLIPper, Piranha) against orthogonally ascertained truth sets — such as GURUG motif sites for TARDBP, 5' splice sites for PRPF8, chrM for FASTKD2, or histone stem-loop motifs for SLBP — by overlap-based TP/FP/FN assignment and precision/recall/F1 computation.

## When to use

Use when comparing called-site outputs from CLIP-seq peak callers (Skipper, CLIPper, Piranha) on eCLIP data and an independent motif-, sequence-, or feature-derived positive truth set is available for the assayed RBP.

## When NOT to use

- No orthogonally derived truth set (motif, splice site, chrM, structural RNA, or affinity-score data) exists for the RBP being evaluated — overlap-based TP/FP/FN assignment cannot be performed
- The goal is peak calling itself rather than benchmarking caller performance; use Skipper's pipeline (GFF partitioning, enriched window identification, fine-mapping) instead
- CLIP datasets lack matched input samples required for Skipper's beta-binomial statistical model, so Skipper output — one side of the comparison — cannot be generated

## Inputs

- Skipper reproducible_enriched_windows tables (from Snakemake/Skipper pipeline on eCLIP reads)
- Called-site peak tables from comparison methods (CLIPper BED output, Piranha)
- RBP-specific orthogonal truth sets (chrM bindings, 5' splice sites, GURUG motif sites, Y RNA genes, histone stem-loop motifs, PUM2 RNA affinity-score-derived sites)

## Outputs

- table_s3_method_benchmark_prf.csv with precision, recall, and F1 for every method-RBP combination across all six RBPs
- Aggregated precision/recall/F1 statistics across the six RBPs (Figure 2G / Table S3 equivalent)

## How to apply

Run Skipper (Snakemake 9.12.0, snakemake-executor-plugin-slurm 1.4.0, STAR alignment with the ENCODE4 single-end protocol, at least 2 CLIP replicates per sample) on the eCLIP datasets for the RBPs of interest to obtain reproducible_enriched_windows tables, and collect called-site tables from the comparison methods (e.g., CLIPper, Piranha). For each RBP define an orthogonal truth set matched to its known biology: chrM binding for FASTKD2 and TROVE2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP, Y RNA genes for TROVE2, and histone stem-loop motifs for SLBP. Mark each called site as true positive, false positive, or false negative by overlap with the RBP-specific truth set, then compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 (harmonic mean) per method-RBP pair plus aggregates across the six RBPs. Assemble the values into a table equivalent to Figure 2G / Table S3 and write it to table_s3_method_benchmark_prf.csv.

## Related tools

- **Skipper** (End-to-end pipeline run via Snakemake to produce reproducible_enriched_windows tables for the six RBPs being benchmarked) — https://github.com/YeoLab/skipper
- **CLIPper** (Existing peak-calling method whose called-site output (BED) is one of the methods evaluated against the truth sets) — https://github.com/YeoLab/clipper
- **Piranha** (Existing peak-calling method whose called-site output is benchmarked against the truth sets alongside Skipper and CLIPper)
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper runs that generate the benchmarked windows)
- **STAR** (Aligner generating the alignments used by Skipper prior to window calling)

## Examples

```
clipper -b YOUR_BAM_FILE.bam -o YOUR_OUT_FILE.bed -s hg19
```

## Evaluation signals

- Output table contains precision, recall, and F1 entries for every method-RBP combination across all six RBPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP)
- All metric values fall in the range [0, 1]
- Each metric is computed from an explicit TP/FP/FN count derived from truth-set overlaps, not imputed or averaged without counts
- Precision, recall, and F1 satisfy the harmonic-mean relationship F1 = 2*P*R/(P+R) within numerical tolerance
- Recomputed table reproduces the Figure 2G / Table S3 pattern: Skipper attains the highest F1 score across all example eCLIPs

## Limitations

- Truth sets are RBP-specific proxies (e.g., chrM, splice sites, motifs) rather than exhaustive true binding sites, so FP/FN assignments depend on truth-set completeness
- Skipper requires a matched input sample to construct the beta-binomial model for statistical testing; CLIP datasets lacking these controls cannot be analyzed and therefore cannot enter this benchmark
- CLIPper output for FASTKD2 and PUM2 were the only cases where Skipper's improvement did not exceed other methods' F1 score by more than 10%, so the expected ranking margin is not uniform across all RBPs

## Evidence

- [methods] Define, for each RBP, an orthogonal positive truth set: chrM binding for FASTKD2 and TROVE2, PUM2 RNA affinity-score-derived sites for PUM2, 5' splice sites for PRPF8, GURUG motif sites for TARDBP: [evidence span withheld — anchor status: not_found]
- [methods] For each method and RBP, mark each called binding site as true positive, false positive, or false negative by overlap with the RBP-specific truth set.: [evidence span withheld — anchor status: not_found]
- [results] Skipper attained the highest F1 score across all example eCLIPs (FASTKD2, PUM2, PRPF8, TARDBP, TROVE2, SLBP); CLIPper output for FASTKD2 and PUM2 were the only cases where Skipper's improvement did not exceed other methods' F1 score by more than 10%.: [evidence span withheld — anchor status: not_found]
- [readme] clipper -b YOUR_BAM_FILE.bam -o YOUR_OUT_FILE.bed -s hg19: [evidence span withheld — anchor status: external_doc]
