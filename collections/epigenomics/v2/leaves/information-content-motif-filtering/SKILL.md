---
name: information-content-motif-filtering
description: Use when filtering CNN-derived motif detectors and their TFBS candidate
  sets from DESSO predict.py output, to discard weak or degenerate motifs before downstream
  motif comparison or TFBS calling.
when_to_use_negative:
- Motif discovery from raw ChIP-seq peaks without trained DESSO CNN detectors — this
  skill only filters and optimizes detector-derived candidate sets
- Ranking motifs by enrichment alone when instance-level TFBS coordinates are needed
  — the binomial model selects a per-motif optimal cutoff, not just a significance
  call
- Genomes/backgrounds other than the GC-matched 101-bp human ENCODE background without
  re-estimating occurrence probabilities, since P-values depend on the 500,000-bin
  background set
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: CNN framework whose predict.py motif-instance outputs (sequence or DNAShape
    motifs) are filtered by the binomial model and information-content retention rules
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:f5d189a60dd6eb994fb128b56e4b72b3bf6f62aa695fb0aecfe7a5fd534aa5f1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OSU-BMBL/DESSO
  repo_url: https://github.com/OSU-BMBL/DESSO
  tools:
  - DESSO
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# information-content-motif-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retention filtering of DESSO-predicted motif detectors using the binomial-distribution TFBS identification model: motifs are kept only if their optimal-cutoff candidate set passes P < 1e-4, contains more than 5 motif instances, and has at least 3 positions with information content > 1.

## When to use

Use when filtering CNN-derived motif detectors and their TFBS candidate sets from DESSO predict.py output, to discard weak or degenerate motifs before downstream motif comparison or TFBS calling.

## When NOT to use

- Motif discovery from raw ChIP-seq peaks without trained DESSO CNN detectors — this skill only filters and optimizes detector-derived candidate sets
- Ranking motifs by enrichment alone when instance-level TFBS coordinates are needed — the binomial model selects a per-motif optimal cutoff, not just a significance call
- Genomes/backgrounds other than the GC-matched 101-bp human ENCODE background without re-estimating occurrence probabilities, since P-values depend on the 500,000-bin background set

## Inputs

- DESSO predict.py motif-instance outputs generated with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
- GC-matched background set of 500,000 101-bp bins (encode_101_background, unpacked into data/)
- Trained DESSO CNN models (output/encode_101/gc_match/<dataset>/Seq/CNN)

## Outputs

- Per-motif, per-cutoff binomial P-value table with pass/fail retention decisions per motif (e.g. binomial_pvalues.tsv)
- Optimal motif instance (TFBS) sets Omega(M, lambda) at the minimum-P-value cutoff for each retained motif
- P-value validation report against an independent binomial-CDF reference implementation (e.g. pvalue_validation_report.tsv)

## How to apply

For each motif M and signal cutoff lambda, build the instance set Omega(M, lambda) of peak sequences whose predicted binding sites pass that cutoff, sweeping cutoffs from start_cutoff 0.01 to end_cutoff 1 with step_cutoff 0.03. Estimate per-motif occurrence probabilities from the GC-matched background of 500,000 101-bp bins (encode_101_background) and compute exact binomial CDF P-values (formula 5) for each Omega(M, lambda). Retain the cutoff corresponding to the minimum P-value as the optimal TFBS set, replacing any subjective motif signal cutoff. Apply the retention filters: keep motifs with P < 1e-4, more than 5 motif instances, and at least 3 positions with information content > 1. Write the per-motif, per-cutoff P-value table with pass/fail decisions (e.g. binomial_pvalues.tsv) and validate computed P-values against an independent binomial-CDF reference implementation.

## Related tools

- **DESSO** (CNN framework whose predict.py motif-instance outputs (sequence or DNAShape motifs) are filtered by the binomial model and information-content retention rules) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Every retained motif satisfies all three filters: P < 1e-4, > 5 motif instances, and ≥ 3 positions with information content > 1
- For each motif, the retained TFBS set corresponds to the cutoff lambda with the minimum binomial P-value across the 0.01–1 sweep (step 0.03)
- Computed P-values match an independent exact binomial CDF reference implementation within recorded deviations (pvalue_validation_report.tsv)
- Occurrence probabilities are estimated from the full 500,000 GC-matched 101-bp background bins, not the peak set itself
- Output table contains one row per motif per cutoff with a binary pass/fail retention decision column

## Limitations

- The binomial P-value depends on background occurrence probabilities estimated from encode_101_background (500,000 101-bp bins); using a different background invalidates the 1e-4 threshold
- Motifs with few instances (>5 required) or low-complexity PWMs (<3 positions with IC > 1) are discarded even if statistically enriched
- DESSO was developed and evaluated on human ENCODE ChIP-seq data (690 datasets, hg19/GRCh37.p13); transfer to other genomes or peak lengths (e.g. 1001-bp GCNN peaks) requires retraining and re-estimating background probabilities

## Evidence

- [other] DESSO's binomial-based statistical model determines the optimal motif instances for each motif detector by computing a binomial P-value for motif instance candidate sets at varying signal cutoffs and selecting the candidates corresponding to the minimum P-value.: [evidence span withheld — anchor status: not_found]
