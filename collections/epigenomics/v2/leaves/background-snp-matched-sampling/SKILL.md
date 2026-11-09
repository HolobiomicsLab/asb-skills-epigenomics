---
name: background-snp-matched-sampling
description: Use when assessing whether a variant set (e.g. GM12878 cell-type-specific
  GTEx v7 eQTLs) carries higher deepC saliency scores than expected for SNPs drawn
  from the same open-chromatin or CTCF-peak regions.
when_to_use_negative:
- The variant set of interest is not localized to open chromatin or CTCF sites, so
  a DNase/CTCF-peak-restricted background would not be region-matched
- Only point statistics (mean/median saliency) are needed and no distributional comparison
  between two samples is intended — a KS test on ECDFs is unnecessary
- No per-SNP saliency track exists yet; the deepC variant-impact prediction step must
  be run first
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pybedtools
  role: Re-sample the matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq
    peak union and intersect eQTL coordinates with saliency tracks
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Read/extract per-SNP saliency scores for eQTL and background SNP positions
    alongside pybedtools
  repo: https://github.com/pysam-developers/pysam
- name: deepC
  role: Source of the per-SNP saliency scores being statistically compared between
    eQTLs and matched background SNPs
  repo: https://github.com/rschwess/deepC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:d418286a2bebe8039c422bbbe003aaa725368015e47ddcfc66c89c516bfa77f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# background-snp-matched-sampling

## Summary

Constructs a matched background SNP set (e.g. re-sampled from ENCODE DNase-seq/CTCF ChIP-seq peak unions via pybedtools) so that deepC saliency scores of eQTLs can be compared to region-matched controls using a two-sample Kolmogorov-Smirnov test, enabling statistical fine-mapping of eQTLs affecting chromatin architecture.

## When to use

Use when assessing whether a variant set (e.g. GM12878 cell-type-specific GTEx v7 eQTLs) carries higher deepC saliency scores than expected for SNPs drawn from the same open-chromatin or CTCF-peak regions.

## When NOT to use

- The variant set of interest is not localized to open chromatin or CTCF sites, so a DNase/CTCF-peak-restricted background would not be region-matched
- Only point statistics (mean/median saliency) are needed and no distributional comparison between two samples is intended — a KS test on ECDFs is unnecessary
- No per-SNP saliency track exists yet; the deepC variant-impact prediction step must be run first

## Inputs

- Per-SNP deepC saliency score tracks from the variant-impact prediction step
- GTEx v7 GM12878/EBV eQTL coordinate list (gtexportal)
- ENCODE DNase-seq / CTCF ChIP-seq peak union (BED-format regions for background sampling)

## Outputs

- Matched background SNP set with extracted saliency scores
- ECDFs of saliency scores for the eQTL set and background set
- ks_test_results. containing the KS statistic D and p-value (expected p < 1e-85)

## How to apply

First intersect the eQTL coordinates (e.g. GTEx v7 GM12878/EBV eQTLs) against per-SNP deepC saliency score tracks produced by the variant-impact prediction step to assign each eQTL a saliency score. Using pybedtools (with a compatible bedtools install), re-sample a matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union so background SNPs come from the same functional regions as the eQTLs, and extract their saliency scores the same way (e.g. with pysam-backed interval lookups). Build empirical cumulative distribution functions (ECDFs) of saliency scores for the eQTL set and the re-sampled background set. Run a two-sample Kolmogorov-Smirnov test (e.g. scipy.stats.ks_2samp) on the two ECDFs to obtain the KS statistic D and its p-value. Write D and the p-value to ks_test_results. and verify the reproduced p-value falls below the paper's reported threshold of 1e-85; a significant result indicates deepC saliency can fine-map eQTLs affecting chromatin architecture.

## Related tools

- **pybedtools** (Re-sample the matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union and intersect eQTL coordinates with saliency tracks) — https://github.com/rschwess/deepC
- **pysam** (Read/extract per-SNP saliency scores for eQTL and background SNP positions alongside pybedtools) — https://github.com/pysam-developers/pysam
- **deepC** (Source of the per-SNP saliency scores being statistically compared between eQTLs and matched background SNPs) — https://github.com/rschwess/deepC

## Examples

```
from scipy.stats import ks_2samp; D, p = ks_2samp(eqtl_saliency_scores, background_saliency_scores); assert p < 1e-85
```

## Evaluation signals

- The reproduced KS p-value falls below the paper's reported threshold of 1e-85
- ks_test_results. contains both a KS statistic D and its p-value as a valid record
- Background SNPs are drawn exclusively from the ENCODE DNase-seq/CTCF ChIP-seq peak union, i.e. every background SNP overlaps a peak-region interval
- eQTL and background saliency scores are extracted via the identical intersection procedure, so the two ECDFs are computed on comparable scores
- ECDFs for both the eQTL set and the re-sampled background set are constructed before running the two-sample test

## Limitations

- The background must be re-sampled from the same regions (ENCODE DNase/CTCF peak union) as the eQTLs; a random genome-wide background would confound regional saliency with functional-region effects
- deepC saliency predictions depend on the installed TensorFlow version (v1.8, 1.14 or 2.1) and a compatible deepC version, and pybedtools requires a compatible version of bedtools installed
- Significance rests on a single two-sample KS test; the finding applies to GM12878 cell-type-specific eQTLs in open chromatin/CTCF sites and was reported with p-value < 1e-85

## Evidence

- [discussion] Fine-mapping with pybedtools/pysam and KS testing rationale: "We demonstrated that deepC can be used to fine-map the sequence determinants of chromatin architecture at base pair resolution and link these with effects on gene expression."
- [methods] pybedtools/pysam installation in methods: "Install additional python library (pysam and pybedtools) using e.g. pip or bioconda"
- [readme] pybedtools and compatible bedtools requirement: "pybedtools and a compatible version of bedtools installed"
