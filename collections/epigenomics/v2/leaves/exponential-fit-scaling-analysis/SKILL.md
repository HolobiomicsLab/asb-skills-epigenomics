---
name: exponential-fit-scaling-analysis
description: Use when analyzing how the spatial auto-correlation of a binned ChIP-seq
  signal changes under coarse-graining, e.g. diagnosing whether broad-domain enrichment
  grows with scale as in H3K27me3 data.
when_to_use_negative:
- When the goal is domain calling output (broadPeak.bed) rather than characterizing
  scale-dependent auto-correlation — run the full RECOGNICER pipeline including domain
  retrieval and significance determination instead
- When the signal is synthetic random-placement data expected to keep lambda near
  100 kb — the fit is still run, but growth of lambda should NOT be expected or claimed
- When C(0) (self-correlation/variance) would be included in the fit — the method
  explicitly ignores C(0) to avoid confounding variance
tools:
- name: RECOGNICER
  role: Performs the recursive block transformation and phase optimization whose intermediate
    coarse-grained maps supply the auto-correlation curves fitted with the exponential
    form
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:f629c607a509325035123fdd79d6accbfd0e564ad7b5cd675555a792a8f2366c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - RECOGNICER
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# exponential-fit-scaling-analysis

## Summary

Fits the two-point auto-correlation curve C(r) ~ C0 * exp(-r/lambda) with a single exponential at each coarse-graining scale to extract a characteristic auto-correlation length lambda, and tracks how lambda scales across recursive block transformation rounds (RECOGNICER). Used to distinguish real ChIP-seq long-range correlation growth from synthetic random-placement data.

## When to use

Use when analyzing how the spatial auto-correlation of a binned ChIP-seq signal changes under coarse-graining, e.g. diagnosing whether broad-domain enrichment grows with scale as in H3K27me3 data.

## When NOT to use

- When the goal is domain calling output (broadPeak.bed) rather than characterizing scale-dependent auto-correlation — run the full RECOGNICER pipeline including domain retrieval and significance determination instead
- When the signal is synthetic random-placement data expected to keep lambda near 100 kb — the fit is still run, but growth of lambda should NOT be expected or claimed
- When C(0) (self-correlation/variance) would be included in the fit — the method explicitly ignores C(0) to avoid confounding variance

## Inputs

- ChIP-seq mapped reads BED file (single-end, e.g. H3K27me3 human CD4+ T cells Chromosome 1)
- Coarse-grained '1'/0 block maps at each recursive transformation round (block factor L=3, majority rule)
- Optional synthetic ChIP-seq control (simulated reads on human Chromosome 1) for comparison

## Outputs

- Table of fitted characteristic auto-correlation length lambda versus coarse-graining scale
- Plot of the growing auto-correlation length trend across scales
- Fitted single-exponential parameters C0 and lambda per coarse-graining round

## How to apply

Bin the chromosome (e.g. human Chromosome 1 from the CD4+ T cell H3K27me3 BED file) into 200 bp non-overlapping windows, shifting single-end reads by half the 150 bp fragment size and marking windows above the random-placement read-count threshold as '1' blocks. Run RECOGNICER's recursive block transformation with block transformation factor L = 3 (majority rule) and phase optimization that selects the phase yielding the most '1' blocks at each round. At each coarse-graining round compute the unnormalized auto-correlation function using only the points C(1) and C(2), deliberately ignoring C(0) to avoid confounding variance. Fit the two-point curve with the single exponential C(r) ~ C0 * exp(-r/lambda) to obtain lambda at each scale, iterating until the termination condition (lambda < 1 on the next coarse scale). The expected behavior is a growing lambda with scale for real H3K27me3 data versus a roughly constant ~100 kb lambda for synthetic random-placement data.

## Related tools

- **RECOGNICER** (Performs the recursive block transformation and phase optimization whose intermediate coarse-grained maps supply the auto-correlation curves fitted with the exponential form) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Output table contains a lambda column with one fitted value per coarse-graining round
- Fitted lambda values grow monotonically with coarse-graining scale for the real H3K27me3 CD4+ T cell dataset (consistent with paper Fig. 3B)
- Synthetic control data yields lambda remaining around 100 kb across scales (Fig. 3A expectation)
- Transformation terminates only when the RECOGNICER termination condition is reached (lambda becomes less than 1 on the next coarse scale)
- Output figure and table files both exist and the plot reproduces the reported growing auto-correlation trend

## Limitations

- RECOGNICER could falsely merge two domains that are separated by gaps on the fine scale into one domain on the coarse scale; individual cases should be monitored by users
- Confounding variance arises if C(0) is included; the fit must use only C(1) and C(2)
- RECOGNICER v1.0 is implemented in Python 2 and requires scipy; parallel runs from the same directory interfere via hard-coded temporary files

## Evidence
