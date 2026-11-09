---
name: poisson-significance-testing-with-fdr
description: Use when candidate enriched domains (e.g. broad H3K27me3 domains from
  coarse-graining) must be filtered for statistical significance against a control
  sample using a local Poisson model with BH FDR correction.
when_to_use_negative:
- The caller needs fine-scale (narrow) peak calling rather than broad-domain significance
  assessment — RECOGNICER is designed for broad domains from kb to Mb
- Only one sample is available with no matched control, since the local Poisson test
  compares ChIP versus control read counts
- A different statistical framework is required (e.g. HMM-based domain calling as
  in RSEG) rather than a per-domain Poisson test
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: RECOGNICER
  role: 'Implements the significance determination step: per-domain ChIP-vs-control
    Poisson test, BH FDR correction, and broadPeak output; invoked with an FDR threshold
    argument'
  repo: https://github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:47f81f9128a505a54e0bd0ead1d445c9cb012d08950eaded20e337eda0b6d70e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# poisson-significance-testing-with-fdr

## Summary

Significance determination step of the RECOGNICER pipeline: each candidate broad domain retrieved by coarse-graining is tested by comparing ChIP versus control read counts under a local Poisson model, with Benjamini-Hochberg correction to control FDR (default 0.01).

## When to use

Use when candidate enriched domains (e.g. broad H3K27me3 domains from coarse-graining) must be filtered for statistical significance against a control sample using a local Poisson model with BH FDR correction.

## When NOT to use

- The caller needs fine-scale (narrow) peak calling rather than broad-domain significance assessment — RECOGNICER is designed for broad domains from kb to Mb
- Only one sample is available with no matched control, since the local Poisson test compares ChIP versus control read counts
- A different statistical framework is required (e.g. HMM-based domain calling as in RSEG) rather than a per-domain Poisson test

## Inputs

- Candidate domains from RECOGNICER coarse-graining/domain retrieval (chrom, start, end) for the ChIP sample
- ChIP-seq mapped reads BED file for the ChIP sample
- ChIP-seq mapped reads BED file for the matched control sample
- FDR threshold argument (e.g. 0.01)

## Outputs

- $SAMPLE-fdr0.01_broadPeak.bed — significant domains in ENCODE broadPeak format (score = fold change, signalValue = ChIP read count)
- $SAMPLE.cgsummary — candidate domains with chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value

## How to apply

After RECOGNICER's domain retrieval step yields candidate domains, compare the ChIP read count and the control read count within each candidate domain, compute a fold-enrichment score and a p-value under a local Poisson model. Apply the Benjamini-Hochberg procedure across all candidate domains to convert p-values into Q-values and control FDR at a chosen threshold (default 0.01, passed as the third argument to RECOGNICER.sh). Keep only domains with FDR (Q-value) at or below the threshold and write them in ENCODE broadPeak format, where the 5th-column score is the ChIP/control fold change and the 7th-column signalValue is the ChIP read count. Validate that the output file parses in broad-Peak format and that every reported domain satisfies the FDR cutoff.

## Related tools

- **RECOGNICER** (Implements the significance determination step: per-domain ChIP-vs-control Poisson test, BH FDR correction, and broadPeak output; invoked with an FDR threshold argument) — https://github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Output file $SAMPLE-fdr0.01_broadPeak.bed exists and parses in ENCODE broadPeak format
- Every reported domain has Q-value (FDR) ≤ 0.01 (or the threshold passed on the command line)
- Each cgsummary row contains all eight fields: chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value
- The broadPeak 5th-column score equals the ChIP/control fold change and the 7th-column signalValue equals the ChIP read count for each domain
- Downsampled runs (e.g. 4M vs 17.23M reads) retain approximately the same aggregate significant-domain length, indicating stability of the significance filter

## Limitations

- RECOGNICER can falsely merge two domains separated by fine-scale gaps into one coarse-scale domain; individual cases should be monitored and featured or masked regions may need to be incorporated
- Passing the FDR filter does not correct for potential arbitrary boundary merges made during coarse-graining — users must inspect individual cases

## Evidence

- [methods] For each candidate domain, compare ChIP versus control read counts, compute a fold-enrichment score and a p-value under a local Poisson model, and apply the Benjamini-Hochberg procedure to control the FDR at 0.01.: [evidence span withheld — anchor status: not_found]
