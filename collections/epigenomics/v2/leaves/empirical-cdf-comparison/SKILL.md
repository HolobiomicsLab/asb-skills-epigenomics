---
name: empirical-cdf-comparison
description: Use when you have per-SNP saliency (or other continuous variant) scores
  for a candidate SNP set plus a matched background set and need to test whether the
  two distributions differ, as in fine-mapping GTEx v7 eQTLs.
when_to_use_negative:
- You only have raw deepC Hi-C interaction predictions and have not yet computed per-SNP
  saliency scores (this skill expects already-scored variants)
- You need to identify TADs or chromatin domains from Hi-C contact matrices — use
  a domain caller such as TopDom instead of a distributional test on saliency scores
- The candidate and background SNP sets are not matched for chromatin context (e.g.
  both should be drawn from DNase/CTCF peak regions), which would confound the KS
  comparison
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Source of the per-SNP saliency score tracks being compared as ECDFs
  repo: https://github.com/rschwess/deepC
- name: pybedtools
  role: Intersects eQTL coordinates with saliency tracks and re-samples matched background
    SNPs from ENCODE DNase/CTCF peak regions
  repo: https://github.com/
- name: pysam
  role: Supports reading genomic coordinate files when extracting saliency scores
    for eQTL and background SNPs
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:3c3310697ccadfde2c2363026bf6382b232bfff5677b0f11e75d814e7c27ac7f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# empirical-cdf-comparison

## Summary

Builds empirical cumulative distribution functions (ECDFs) of deepC saliency scores for a cell-type-specific eQTL set versus a matched, re-sampled background SNP set and compares them with a two-sample Kolmogorov-Smirnov test to judge whether variant saliency is enriched.

## When to use

Use when you have per-SNP saliency (or other continuous variant) scores for a candidate SNP set plus a matched background set and need to test whether the two distributions differ, as in fine-mapping GTEx v7 eQTLs.

## When NOT to use

- You only have raw deepC Hi-C interaction predictions and have not yet computed per-SNP saliency scores (this skill expects already-scored variants)
- You need to identify TADs or chromatin domains from Hi-C contact matrices — use a domain caller such as TopDom instead of a distributional test on saliency scores
- The candidate and background SNP sets are not matched for chromatin context (e.g. both should be drawn from DNase/CTCF peak regions), which would confound the KS comparison

## Inputs

- Per-SNP deepC saliency score tracks from the variant-impact prediction step
- GTEx v7 GM12878/EBV eQTL coordinates (gtexportal)
- ENCODE DNase-seq / CTCF ChIP-seq peak union (BED) for background re-sampling

## Outputs

- Saliency scores assigned to each cell-type-specific eQTL
- Matched background SNP set with saliency scores
- ks_test_results.json containing the KS statistic D and p-value

## How to apply

Load the per-SNP saliency score tracks from the deepC variant-impact prediction step and intersect the GTEx v7 GM12878/EBV eQTL coordinates against them so every eQTL receives a saliency score. Using pybedtools, re-sample a matched background SNP set from the union of ENCODE DNase-seq/CTCF ChIP-seq peaks and extract background saliency scores the same way, so both sets come from comparable chromatin contexts. Compute the ECDFs of saliency scores for the eQTL and background sets and run a two-sample Kolmogorov-Smirnov test (scipy.stats.ks_2samp) to obtain the KS statistic D and its p-value. Write D and the p-value to ks_test_results.json and treat the analysis as successful only when the reproduced p-value falls below the paper's reported threshold of 1e-85, indicating deepC saliency fine-maps eQTLs affecting chromatin architecture.

## Related tools

- **deepC** (Source of the per-SNP saliency score tracks being compared as ECDFs) — https://github.com/rschwess/deepC
- **pybedtools** (Intersects eQTL coordinates with saliency tracks and re-samples matched background SNPs from ENCODE DNase/CTCF peak regions) — https://github.com/
- **pysam** (Supports reading genomic coordinate files when extracting saliency scores for eQTL and background SNPs) — https://github.com/pysam-developers/pysam

## Examples

```
from scipy.stats import ks_2samp; D, p = ks_2samp(eqtl_saliency, background_saliency)
```

## Evaluation signals

- Reproduced p-value falls below the paper's reported threshold of 1e-85 for the GM12878 eQTL vs. re-sampled SNP comparison
- ks_test_results.json contains both the KS statistic D and a p-value with the expected schema
- Every eQTL coordinate in the GTEx v7 set is assigned exactly one saliency score from the deepC tracks
- Background SNPs re-sampled via pybedtools overlap the union of ENCODE DNase-seq/CTCF ChIP-seq peaks, matching the regions the eQTLs were intersected against
- Empirical CDFs for both sets are computable (non-empty, finite saliency score arrays) before calling scipy.stats.ks_2samp

## Limitations

- The KS result depends on the background re-sampling scheme; SNPs must be drawn from the same open-chromatin/CTCF peak union to be a fair comparison
- pybedtools requires a compatible version of bedtools installed alongside it, and deepC requires specific supported TensorFlow versions (v1.8, 1.14, 2.1; other versions untested)
- A significant p-value only indicates distributional difference in saliency, not a direct deepC-vs-Akita benchmark; the authors defer comparative consolidation between such models to future work

## Evidence
