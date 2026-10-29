---
name: multiscale-genomic-signal-analysis
description: Use when a ChIP-seq signal (e.g. H3K27me3 in human CD4+ T cells, chromosome
  1) must be analyzed across multiple length scales to measure how its spatial auto-correlation
  length evolves under recursive coarse-graining.
when_to_use_negative:
- The goal is standard sharp-peak calling with a local statistical model rather than
  multi-scale broad-domain or scale-dependence analysis
- The input is already a called peak/domain list or feature table rather than raw
  mapped reads in BED format
- Only a single-scale enrichment profile is needed and the recursive coarse-graining/auto-correlation
  machinery would add no information
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RECOGNICER
  role: Performs the recursive block transformation (L = 3 majority rule) with phase
    optimization and termination control, providing the coarse-grained block maps
    on which auto-correlation lengths are computed at each scale
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:071d007b00b8fa95a647de14d22f10937f9b661b77307313b82ef01101e553ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multiscale-genomic-signal-analysis

## Summary

Characterize genomic ChIP-seq signal across length scales by applying RECOGNICER's recursive block transformation with phase optimization and fitting the auto-correlation function C(r) ~ C0 * exp(-r/lambda) at each coarse-graining round, to reveal scale-dependent structure (e.g. growing auto-correlation length in real H3K27me3 data vs ~100 kb in synthetic data).

## When to use

Use when a ChIP-seq signal (e.g. H3K27me3 in human CD4+ T cells, chromosome 1) must be analyzed across multiple length scales to measure how its spatial auto-correlation length evolves under recursive coarse-graining.

## When NOT to use

- The goal is standard sharp-peak calling with a local statistical model rather than multi-scale broad-domain or scale-dependence analysis
- The input is already a called peak/domain list or feature table rather than raw mapped reads in BED format
- Only a single-scale enrichment profile is needed and the recursive coarse-graining/auto-correlation machinery would add no information

## Inputs

- ChIP-seq mapped reads BED file (e.g. H3K27me3 ChIP-seq for human CD4+ T cells, chromosome 1)
- Control sample mapped reads BED file (RECOGNICER input)
- Parameters: window size 200 bp, fragment size 150 bp, block transformation factor L = 3, FDR threshold (e.g. 0.01)

## Outputs

- Table of characteristic auto-correlation length lambda versus coarse-graining scale
- Plot of fitted lambda trend across coarse-graining rounds
- RECOGNICER coarse-grained block maps and $SAMPLE.cgsummary candidate domain summary

## How to apply

Bin the chromosome (e.g. human chromosome 1) into 200 bp non-overlapping windows, shifting each single-end read from its 5' end by half the 150 bp default DNA fragment size, and designate windows above the random-placement read-count threshold as '1' blocks. Run RECOGNICER's recursive block transformation with block transformation factor L = 3 (majority rule), applying the phase optimization that selects at each round the phase yielding the most '1' blocks on the coarse scale. At each coarse-graining round, compute the unnormalized auto-correlation function using only the points C(1) and C(2), ignoring C(0) to avoid confounding variance, and fit the two-point curve with the single exponential form C(r) ~ C0 * exp(-r/lambda) to obtain the characteristic auto-correlation length lambda. Continue until RECOGNICER's termination condition is reached (lambda would become less than 1 on the next coarse scale), recording lambda at every scale, then tabulate and plot lambda versus coarse-graining scale.

## Related tools

- **RECOGNICER** (Performs the recursive block transformation (L = 3 majority rule) with phase optimization and termination control, providing the coarse-grained block maps on which auto-correlation lengths are computed at each scale) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Output table contains a lambda (auto-correlation length) column with one fitted value per coarse-graining round
- For real H3K27me3 CD4+ T cell data, fitted lambda values grow monotonically with the coarse-graining scale (paper Fig. 3B)
- For the synthetic dataset (simulated reads on human chromosome 1), lambda remains around 100 kb across scales (paper Fig. 3A) — a negative control
- The run terminates under the RECOGNICER condition that lambda becomes less than 1 on the next coarse scale
- Identified H3K27me3 domain sizes range from kb to Mb (paper Fig. 3C)

## Limitations

- RECOGNICER may falsely merge two domains separated by fine-scale gaps into one coarse-scale domain; such cases should be monitored by users, who can incorporate featured or masked regions
- RECOGNICER v1.0 is implemented in python2 and requires the scipy package; do not run multiple RECOGNICER instances from the same directory in parallel due to hard-coded temporary file names
- Auto-correlation fit must ignore C(0) to avoid confounding variance; using it distorts the exponential fit

## Evidence
