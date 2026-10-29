---
name: eqtl-saliency-enrichment-analysis
description: Use when you have per-SNP deepC variant-impact/saliency score tracks
  and want to test whether a candidate SNP set (e.g. cell-type-specific eQTLs) is
  enriched for high saliency against a re-sampled, region-matched background via a
  two-sample KS test.
when_to_use_negative:
- You have not yet generated per-SNP saliency scores with deepC — run the variant-impact
  prediction step first, since this skill only re-analyzes existing score tracks.
- Your candidate variant set is not spatially restricted to open chromatin or CTCF
  sites, so the ENCODE peak-union background would not be a matched null.
- You need a direct deepC-vs-Akita comparison — the paper explicitly presents no such
  benchmark, deferring comparative study to future work.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Produces the per-SNP variant-impact/saliency score tracks that are intersected
    with eQTL coordinates; imported in Python as deepCregr
  repo: github.com/rschwess/deepC
- name: pybedtools
  role: Re-samples the matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq
    peak union
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Supports handling of SNP/coordinate data when extracting saliency scores for
    eQTLs and background SNPs
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:ce4baa866e44bace4c7177103bb3a6ba4594dac5f61061bc7131c2b3ac1a8402
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# eqtl-saliency-enrichment-analysis

## Summary

Tests whether cell-type-specific eQTLs (e.g. GTEx v7 GM12878) have elevated deepC saliency scores relative to randomly re-sampled SNPs from matched open-chromatin/CTCF regions, using a two-sample Kolmogorov-Smirnov test. This reproduces the deepC paper's finding that saliency can fine-map sequence determinants of chromatin architecture linked to gene expression.

## When to use

Use when you have per-SNP deepC variant-impact/saliency score tracks and want to test whether a candidate SNP set (e.g. cell-type-specific eQTLs) is enriched for high saliency against a re-sampled, region-matched background via a two-sample KS test.

## When NOT to use

- You have not yet generated per-SNP saliency scores with deepC — run the variant-impact prediction step first, since this skill only re-analyzes existing score tracks.
- Your candidate variant set is not spatially restricted to open chromatin or CTCF sites, so the ENCODE peak-union background would not be a matched null.
- You need a direct deepC-vs-Akita comparison — the paper explicitly presents no such benchmark, deferring comparative study to future work.

## Inputs

- Per-SNP deepC saliency/variant-impact score tracks (output of the deepC variant-impact prediction step)
- GTEx v7 GM12878/EBV eQTL coordinates from gtexportal
- ENCODE DNase-seq / CTCF ChIP-seq peak union (BED) used for background re-sampling
- Reference genome FASTA accessible via pysam for extracting SNP contexts

## Outputs

- ks_test_results. containing the KS statistic D and p-value
- ECDFs of saliency scores for the eQTL set and the re-sampled background set
- Table of per-eQTL saliency scores from coordinate intersection

## How to apply

Intersect the GTEx v7 GM12878/EBV eQTL coordinates (gtexportal) against the per-SNP saliency score tracks produced by the deepC variant-impact prediction step to assign each eQTL a saliency score. Using pybedtools, re-sample a matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union and extract background saliency scores identically. Build empirical cumulative distribution functions (ECDFs) for both sets and run scipy.stats.ks_2samp to obtain the KS statistic D and p-value. Write D and the p-value to ks_test_results. and verify the reproduced p-value falls below the paper's reported threshold of 1e-85; a significant result indicates deepC saliency can fine-map eQTLs affecting chromatin architecture.

## Related tools

- **deepC** (Produces the per-SNP variant-impact/saliency score tracks that are intersected with eQTL coordinates; imported in Python as deepCregr) — github.com/rschwess/deepC
- **pybedtools** (Re-samples the matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union) — https://github.com/rschwess/deepC
- **pysam** (Supports handling of SNP/coordinate data when extracting saliency scores for eQTLs and background SNPs) — https://github.com/pysam-developers/pysam

## Examples

```
from scipy.stats import ks_2samp; D, p = ks_2samp(eqtl_saliency_scores, background_saliency_scores)
```

## Evaluation signals

- Reproduced p-value from scipy.stats.ks_2samp falls below the paper's reported threshold of 1e-85
- ks_test_results. contains both the KS statistic D and a numeric p-value
- Background SNPs are drawn from the same ENCODE DNase-seq/CTCF ChIP-seq peak union regions as the eQTLs (region-matched null)
- ECDFs exist for both the eQTL set and the background set before the KS test is run
- Every eQTL in the GTEx v7 GM12878/EBV set obtained a saliency score from the intersection with the per-SNP tracks

## Limitations

- The significance claim is specific to GM12878 cell-type-specific eQTLs in open chromatin or CTCF sites; other cell types or variant classes are not covered by this reproduction.
- deepC was developed under tensorflow v1.8 and supports only v1.8, 1.14 and 2.1 — other versions are untested, so the upstream saliency predictions depend on a compatible tensorflow install.
- pybedtools requires a compatible version of the bedtools binary to be installed alongside the Python package.

## Evidence

- [other] GM12878 eQTL enrichment finding: [evidence span withheld — anchor status: not_found]
