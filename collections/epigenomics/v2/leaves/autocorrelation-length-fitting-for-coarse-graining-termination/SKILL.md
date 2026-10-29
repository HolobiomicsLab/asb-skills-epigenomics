---
name: autocorrelation-length-fitting-for-coarse-graining-termination
description: Use when running RECOGNICER coarse-graining on binned ChIP-seq read counts
  and you need an objective, scale-aware criterion to terminate the recursive block
  transformation and proceed to domain retrieval.
when_to_use_negative:
- The analysis goal is sharp, local peak calling without multi-scale structure — RECOGNICER's
  coarse-graining is designed for broad domains (kb to Mb, e.g. H3K27me3/H3K9me3),
  and existing local-statistical peak callers are cited as the alternative.
- Running multiple RECOGNICER instances from the same directory — temporary files
  have hard-coded names and parallel runs would interfere with the coarse-graining
  and termination state.
- Genome version not in GenomeData.py (e.g. unsupported species) without first adding
  chromosome lists and lengths — the window partitioning that feeds the auto-correlation
  computation depends on genome metadata.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: RECOGNICER
  role: Implements the recursive block transformation whose per-round auto-correlation
    curve and fitted characteristic length lambda drive the coarse-graining termination
    criterion
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:58c17688720ee4300260036abf441ae9a39b43c32b0d6fd19195f8669bc084a7
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# autocorrelation-length-fitting-for-coarse-graining-termination

## Summary

Fits a single exponential C(r) ~ C0 * e^(-r/lambda) to the auto-correlation curve of the coarse-grained ChIP-seq block map at each RECOGNICER coarse-graining round; the fitted characteristic auto-correlation length (lambda, in bp) is used to decide when to stop the recursive block transformation.

## When to use

Use when running RECOGNICER coarse-graining on binned ChIP-seq read counts and you need an objective, scale-aware criterion to terminate the recursive block transformation and proceed to domain retrieval.

## When NOT to use

- The analysis goal is sharp, local peak calling without multi-scale structure — RECOGNICER's coarse-graining is designed for broad domains (kb to Mb, e.g. H3K27me3/H3K9me3), and existing local-statistical peak callers are cited as the alternative.
- Running multiple RECOGNICER instances from the same directory — temporary files have hard-coded names and parallel runs would interfere with the coarse-graining and termination state.
- Genome version not in GenomeData.py (e.g. unsupported species) without first adding chromosome lists and lengths — the window partitioning that feeds the auto-correlation computation depends on genome metadata.

## Inputs

- ChIP and control mapped-read BED files partitioned into 200 bp non-overlapping windows with reads shifted 75 bp from the 5' end (150 bp mono-nucleosomal fragment)
- Binary ('0'/'1') block maps at each coarse-graining scale produced by recursive L=3 majority-rule block transformations with phase optimization

## Outputs

- Fitted characteristic auto-correlation length lambda (in bp or blocks) per coarse-graining round
- Coarse-graining termination decision marking the coarsest scale for domain trace-back
- Candidate domain list carried into significance determination and written to $SAMPLE.cgsummary and $SAMPLE-fdr0.01_broadPeak.bed

## How to apply

During each recursive block transformation round (block transformation factor L=3, majority rule, phase optimization selecting the phase with the most '1' blocks), compute the auto-correlation curve on the coarse-scale block map. At the coarsest and previous scales compute only the auto-correlation values C(1) and C(2), then fit the characteristic length lambda with a single exponential C(r) ~ C0 * e^(-r/lambda). Terminate coarse-graining when lambda at both the present and previous scales exceeds 1 block, i.e., when lambda would fall below 1 on the next coarse scale; this means the signal has lost spatial coherence and further coarse-graining is unnecessary. Rationale: synthetic data with ~100 kb domains keeps lambda near 100 kb across scales, while real H3K27me3 data show a growing auto-correlation length with coarse-graining, so lambda tracks the scale of clustered enriched elements. After termination, trace back from the coarsest to the finest scale to retrieve candidate domains before significance determination (local Poisson model, Benjamini-Hochberg FDR at 0.01).

## Related tools

- **RECOGNICER** (Implements the recursive block transformation whose per-round auto-correlation curve and fitted characteristic length lambda drive the coarse-graining termination criterion) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- The fitted lambda at each recorded round is positive and exceeds 1 block exactly at the present and previous scales where coarse-graining is declared finished.
- For synthetic data with known ~100 kb domains, the auto-correlation length remains around 100 kb across coarse-graining scales (Fig. 3A expectation).
- Coarse-graining rounds stop before lambda would drop below 1 block on the next coarse scale, and no rounds are skipped in the trace-back to the finest 200 bp scale.
- The final $SAMPLE-fdr0.01_broadPeak.bed exists, parses in ENCODE broadPeak format, and all reported domains satisfy FDR <= 0.01 (Q-value column in $SAMPLE.cgsummary).
- Identified H3K27me3 domain sizes span kb to Mb, consistent with multi-scale detection reported for human CD4+ T cell data.

## Limitations

- RECOGNICER could falsely merge two domains that are separated by gaps on the fine scale into one domain on the coarse scale; such cases should be monitored by users, who can incorporate featured or masked regions.
- Termination relies on only C(1) and C(2) at the coarsest scales, so the exponential fit is a coarse approximation of the auto-correlation decay.
- v1.0 is implemented in Python 2 and requires the scipy package; whole-process runtime may take several hours, and parallel instances from one directory corrupt temporary files.

## Evidence

- [results] At each coarse-graining round, the auto-correlation curve and fitted length drive termination: "At each around of the recursive block transformation, we compute the auto-correlation curve and plot the fitted characteristic auto-correlation length"
- [results] Synthetic-data sanity check: lambda stays near the true domain scale of ~100 kb: "We show that the auto-correlation length remains around 100 kb for the synthetic data, as expected (Fig. 3A)"
- [methods] The three-step pipeline containing the coarse-graining termination decision point: "The RECOGNICER algorithm includes three main steps: coarse-graining by recursive block transformation, candidate domain retrieval with boundary determination, and significance determination"
