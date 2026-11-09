---
name: coarse-graining-block-transformation
description: Use when calling broad ChIP-seq enrichment domains (e.g., H3K27me3/H3K9me3
  spanning kb to Mb) from mapped-read BED files, or when characterizing multi-scale
  structure via auto-correlation length under coarse-graining.
when_to_use_negative:
- The goal is sharp/narrow peak calling for point-source enrichment (e.g., transcription
  factor binding), since RECOGNICER targets broad domains from kb to Mb
- Only a ChIP sample is available without a control sample, as significance determination
  compares ChIP versus control read counts under a local Poisson model
- Multiple RECOGNICER instances must share one working directory, since hard-coded
  temporary file names would interfere and produce absurd outcomes
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: RECOGNICER
  role: Implements the recursive block transformation coarse-graining with L=3 majority
    rule, phase optimization, auto-correlation-length fitting, and termination criterion;
    driven by RECOGNICER.sh
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:4b6da9e3deff0a9d5bfccc5e05ceaf015cfc784937cdccc76a6d89f86688f152
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# coarse-graining-block-transformation

## Summary

Recursive coarse-graining via block transformation (factor L=3 with majority rule and phase optimization) that determines spatial clustering of enriched ChIP-seq windows across multiple length scales, terminating when the fitted auto-correlation length lambda drops below 1 on the next coarse scale. It is the first core step of RECOGNICER's three-step pipeline for broad domain identification.

## When to use

Use when calling broad ChIP-seq enrichment domains (e.g., H3K27me3/H3K9me3 spanning kb to Mb) from mapped-read BED files, or when characterizing multi-scale structure via auto-correlation length under coarse-graining.

## When NOT to use

- The goal is sharp/narrow peak calling for point-source enrichment (e.g., transcription factor binding), since RECOGNICER targets broad domains from kb to Mb
- Only a ChIP sample is available without a control sample, as significance determination compares ChIP versus control read counts under a local Poisson model
- Multiple RECOGNICER instances must share one working directory, since hard-coded temporary file names would interfere and produce absurd outcomes

## Inputs

- ChIP-seq mapped reads in BED format (ChIP sample)
- Control sample mapped reads in BED format
- Configuration parameters in RECOGNICER.sh: SAMPLE name, SPECIES/genome version, OUT_DIR, WINDOW_SIZE (default 200), FRAGMENT_SIZE (default 150)

## Outputs

- $SAMPLE.cgsummary: coarse-graining summary of all candidate domains (chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value)
- $SAMPLE-fdr0.01_broadPeak.bed: significant domains in ENCODE broadPeak format
- Auto-correlation length lambda versus coarse-graining scale table/curve (C(r) ~ C0 * exp(-r/lambda) fits)
- $SAMPLE-W200.bedgraph: read pile-up track at window-size resolution for visualization

## How to apply

Partition each chromosome into equal-sized non-overlapping 200 bp windows (WINDOW_SIZE default 200 bp, mononucleosome+linker), shift single-end read 5' coordinates downstream by half the FRAGMENT_SIZE (75 bp for the default 150 bp fragment), and designate windows with read counts above the random-placement average threshold as '1' blocks. Perform recursive block transformations with block transformation factor L = 3 using the majority rule, and in each round run all L = 3 possible phases, selecting the phase that yields the largest number of '1' blocks on the coarse scale to avoid phase arbitrariness. At each round compute the auto-correlation curve using only C(1) and C(2) (ignoring C(0)) and fit the single exponential C(r) ~ C0 * e^(-r/lambda) to obtain the characteristic length lambda. Terminate when lambda at both the present and previous scales exceeds 1 (i.e., lambda < 1 on the next coarse scale), then hand candidate blocks to domain retrieval with boundary determination and Poisson-based significance determination with Benjamini-Hochberg FDR control (threshold 0.01 in RECOGNICER.sh).

## Related tools

- **RECOGNICER** (Implements the recursive block transformation coarse-graining with L=3 majority rule, phase optimization, auto-correlation-length fitting, and termination criterion; driven by RECOGNICER.sh) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- For synthetic data the fitted lambda stays around 100 kb across coarse-graining scales; for real H3K27me3 ChIP-seq data lambda grows with coarse-graining (Fig. 3A/3B)
- Coarse-graining terminates exactly when lambda becomes less than 1 on the next coarse scale (lambda at present and previous scales exceeding 1)
- $SAMPLE-fdr0.01_broadPeak.bed exists and parses in ENCODE broadPeak format with all domains satisfying FDR <= 0.01
- $SAMPLE.cgsummary exists with the 8-column schema (chrom, start, end, ChIP count, control count, P-value, fold change, Q-value)
- At every round the selected phase is the one yielding the largest number of '1' blocks on the coarse scale

## Limitations

- RECOGNICER could falsely merge two domains that are separated by gaps on the fine scale into one domain on the coarse scale; such individual cases should be monitored by users, who can incorporate featured or masked regions
- It is strongly NOT recommended to change parameters other than WINDOW_SIZE, FRAGMENT_SIZE, GENOME_FRACTION, and CHIPTHRESHOLD
- Parallel RECOGNICER instances from the same directory interfere via hard-coded temporary file names, producing absurd outcomes
- RECOGNICER v1.0 is implemented in python2 and requires the scipy package

## Evidence
