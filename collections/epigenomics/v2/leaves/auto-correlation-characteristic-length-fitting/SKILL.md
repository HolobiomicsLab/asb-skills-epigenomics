---
name: auto-correlation-characteristic-length-fitting
description: 'Use when diagnosing multi-scale spatial structure in binned ChIP-seq
  data: compute and fit the auto-correlation curve at each coarse-graining scale to
  distinguish real broad-domain signal (growing lambda) from synthetic/random-placement
  data (flat lambda ~100 kb).'
when_to_use_negative:
- When the goal is simply calling broad domains rather than characterizing multi-scale
  correlation structure — run RECOGNICER's domain retrieval and significance determination
  instead
- When the input is already coarse-grained binary data without known bin size or fragment-shift
  provenance, since the 200 bp window / 150 bp fragment shift preprocessing is integral
  to the lambda estimate
- For narrow-peak ChIP-seq signals, where broad-domain multi-scale correlation behavior
  is not the expected regime
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3520
tools:
- name: RECOGNICER
  role: Performs the recursive block transformation (L = 3, majority rule, phase optimization)
    whose per-round output feeds the auto-correlation computation and exponential
    lambda fit; terminates when lambda < 1 on the next coarse scale
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:f18b1d8ff1b1c501fbd8099849b0eaaf621a93fc04e5c6b0927cd6af6537a2bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# auto-correlation-characteristic-length-fitting

## Summary

Fits the characteristic auto-correlation length (lambda) of ChIP-seq signal at each round of RECOGNICER's recursive coarse-graining block transformation by fitting the two-point auto-correlation curve to a single exponential C(r) ~ C0 * exp(-r/lambda), to quantify how spatial correlation structure changes across scales.

## When to use

Use when diagnosing multi-scale spatial structure in binned ChIP-seq data: compute and fit the auto-correlation curve at each coarse-graining scale to distinguish real broad-domain signal (growing lambda) from synthetic/random-placement data (flat lambda ~100 kb).

## When NOT to use

- When the goal is simply calling broad domains rather than characterizing multi-scale correlation structure — run RECOGNICER's domain retrieval and significance determination instead
- When the input is already coarse-grained binary data without known bin size or fragment-shift provenance, since the 200 bp window / 150 bp fragment shift preprocessing is integral to the lambda estimate
- For narrow-peak ChIP-seq signals, where broad-domain multi-scale correlation behavior is not the expected regime

## Inputs

- ChIP-seq mapped reads BED file for the ChIP signal sample (e.g., H3K27me3 ChIP-seq for human CD4+ T cells, Chromosome 1)
- Control sample mapped reads BED file (RECOGNICER also requires a control input)
- Binning parameters: WINDOW_SIZE = 200 bp, FRAGMENT_SIZE = 150 bp, block transformation factor L = 3

## Outputs

- Table of characteristic auto-correlation length (lambda) versus coarse-graining round/scale
- Plot of lambda versus coarse-graining scale showing the growing auto-correlation trend for real data
- Per-round auto-correlation curves C(1), C(2) used for the exponential fit

## How to apply

First bin the chromosome (e.g., human Chromosome 1 H3K27me3 ChIP-seq for CD4+ T cells) into 200 bp non-overlapping windows, shifting each single-end read from its 5' end by half of the 150 bp default DNA fragment size, and designate windows above the random-placement read-count threshold as '1' blocks. Then run RECOGNICER's recursive block transformation with block transformation factor L = 3 (majority rule) plus the phase optimization that selects, at each round, the phase yielding the most '1' blocks on the coarse scale. At each coarse-graining round compute the unnormalized auto-correlation function using only points C(1) and C(2), deliberately ignoring C(0) to avoid confounding from variance, and fit the two-point auto-correlation curve with the single exponential form C(r) ~ C0 * exp(-r/lambda) to obtain the characteristic auto-correlation length lambda at that scale. Continue until the termination condition is reached (lambda would become less than 1 on the next coarse scale), recording lambda at every scale, then tabulate and plot lambda versus coarse-graining scale. For real data the fitted lambda should grow with coarse-graining scale, whereas synthetic data with random background placement keeps lambda around 100 kb.

## Related tools

- **RECOGNICER** (Performs the recursive block transformation (L = 3, majority rule, phase optimization) whose per-round output feeds the auto-correlation computation and exponential lambda fit; terminates when lambda < 1 on the next coarse scale) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Output table exists with a lambda column recording a value at every coarse-graining round
- Fitted lambda values grow monotonically (or near-monotonically) with the coarse-graining scale for the real H3K27me3 dataset, matching the paper's Fig. 3B finding
- As a negative control, synthetic data (simulated reads on human Chromosome 1) keep lambda around 100 kb across scales, matching Fig. 3A
- Auto-correlation fit uses only C(1) and C(2); C(0) is excluded from the fit to avoid variance confounding
- Transformation stops when the RECOGNICER termination condition (lambda < 1 on the next coarse scale) is reached

## Limitations

- C(0) is excluded from the fit because it is confounded by variance; relying on it would distort lambda
- RECOGNICER could falsely merge two domains separated by gaps on the fine scale into one domain on the coarse scale, which affects coarse-scale lambda
- RECOGNICER v1.0 is implemented in Python 2 and requires the scipy package; temporary files have hard-coded names, so parallel instances must run in separate directories
- The random-placement read-count threshold designation of '1' blocks depends on the chosen window size (200 bp) and fragment size (150 bp) defaults

## Evidence

- [results] Auto-correlation fit uses only two points: "At each around of the recursive block transformation, we compute the auto-correlation curve and plot the fitted characteristic auto-correlation length"
- [results] Synthetic-data invariant for lambda: "We show that the auto-correlation length remains around 100 kb for the synthetic data, as expected (Fig. 3A)"
- [results] Real-data growth of lambda: "the real ChIP-seq experimental data exhibit a growing auto-correlation along with coarse-graining (Fig. 3B)"
- [methods] Recursive block transformation is the core coarse-graining engine: "The RECOGNICER algorithm includes three main steps: coarse-graining by recursive block transformation, candidate domain retrieval with boundary determination, and significance determination"
- [methods] Reads are placed into 200 bp non-overlapping windows after processing: "To process a ChIP-seq dataset, we first partition each chromosome in the genome into equal-sized non-overlapping windows. We place mapped ChIP-seq reads into the windows."
- [readme] Fragment-size shift parameter grounding: "FRAGMENT_SIZE`: ChIP DNA fragment size, used for determination of the shift size from the 5' start location of a sequence read towards the center of the DNA fragment represented by the read"
