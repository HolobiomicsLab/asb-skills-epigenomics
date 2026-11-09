---
name: two-sample-kolmogorov-smirnov-testing
description: Use when two independent score distributions (e.g. deepC saliency scores
  for eQTLs vs re-sampled matched background SNPs) must be compared non-parametrically
  via ECDFs to obtain a KS statistic D and p-value.
when_to_use_negative:
- The two score sets are paired or not independent samples (e.g. same SNPs scored
  under two models) — a paired test is the correct choice instead.
- You need an effect size or fold-change of saliency rather than a whole-distribution
  difference; the KS test only reports maximum ECDF separation (D).
- Input is deepC raw interaction predictions rather than the derived per-SNP saliency
  scores — the prediction step must run first.
tools:
- name: pybedtools
  role: Intersect eQTL coordinates with saliency tracks and re-sample the matched
    background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Python library installed alongside pybedtools to support the bed/genome interval
    handling used to assign saliency scores to SNPs
  repo: https://github.com/pysam-developers/pysam
- name: deepC
  role: Produces the upstream per-SNP saliency score tracks that the KS test compares
  repo: https://github.com/rschwess/deepC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:ee3d4e7fef2492e292848402d11327ec5f6966dfd0c1495e5b1e669ccfec56d9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# two-sample-kolmogorov-smirnov-testing

## Summary

Non-parametric two-sample Kolmogorov–Smirnov test comparing empirical cumulative distribution functions of deepC per-SNP saliency scores between GTEx v7 GM12878 cell-type-specific eQTLs and matched re-sampled background SNPs, to test whether eQTLs in open chromatin/CTCF sites carry higher saliency.

## When to use

Use when two independent score distributions (e.g. deepC saliency scores for eQTLs vs re-sampled matched background SNPs) must be compared non-parametrically via ECDFs to obtain a KS statistic D and p-value.

## When NOT to use

- The two score sets are paired or not independent samples (e.g. same SNPs scored under two models) — a paired test is the correct choice instead.
- You need an effect size or fold-change of saliency rather than a whole-distribution difference; the KS test only reports maximum ECDF separation (D).
- Input is deepC raw interaction predictions rather than the derived per-SNP saliency scores — the prediction step must run first.

## Inputs

- Per-SNP deepC saliency score tracks from the variant-impact prediction step
- GTEx v7 GM12878/EBV eQTL coordinates (gtexportal)
- ENCODE DNase-seq / CTCF ChIP-seq peak union (BED) for background re-sampling
- Re-sampled matched background SNP set with saliency scores

## Outputs

- ks_test_results. containing the KS statistic D and p-value
- ECDFs of saliency scores for the eQTL set and the re-sampled background set

## How to apply

Load the per-SNP saliency score tracks from the deepC variant-impact prediction step and intersect GTEx v7 GM12878/EBV eQTL coordinates against them to assign each eQTL a saliency score. Using pybedtools, re-sample a matched background SNP set from the union of ENCODE DNase-seq and CTCF ChIP-seq peaks and extract background saliency scores the same way, so both sets are drawn from comparable functional regions. Build the empirical cumulative distribution functions (ECDFs) of the two score sets, then run a two-sample Kolmogorov–Smirnov test (scipy.stats.ks_2samp) to obtain the KS statistic D and its p-value. Write D and the p-value to ks_test_results. and check reproduction against the paper's reported threshold of p < 1e-85. The non-parametric KS comparison is appropriate because it makes no distributional assumptions and directly quantifies the maximum separation between the two ECDFs.

## Related tools

- **pybedtools** (Intersect eQTL coordinates with saliency tracks and re-sample the matched background SNP set from the ENCODE DNase-seq/CTCF ChIP-seq peak union) — https://github.com/rschwess/deepC
- **pysam** (Python library installed alongside pybedtools to support the bed/genome interval handling used to assign saliency scores to SNPs) — https://github.com/pysam-developers/pysam
- **deepC** (Produces the upstream per-SNP saliency score tracks that the KS test compares) — https://github.com/rschwess/deepC

## Examples

```
from scipy.stats import ks_2samp; D, p = ks_2samp(eqtl_saliency_scores, background_saliency_scores)
```

## Evaluation signals

- Reproduced p-value falls below the paper's reported threshold of 1e-85 (finding: eQTLs have significantly higher saliency than re-sampled background SNPs).
- KS statistic D lies in [0, 1] and the eQTL ECDF lies to the right of (dominates) the background ECDF across the score range.
- Background SNP set is drawn from the same ENCODE DNase-seq/CTCF ChIP-seq peak union regions as the eQTLs (matched sampling, via pybedtools).
- ks_test_results. is a valid record containing both the KS statistic D and the p-value.
- Every eQTL in the GTEx v7 GM12878/EBV set receives exactly one assigned saliency score after intersection with the score tracks.

## Limitations

- The KS p-value depends on background re-sampling; a different matched background draw can shift D and the p-value.
- The deepC saliency scores feeding the test depend on tensorflow version compatibility (v1.8, 1.14 or 2.1) and pybedtools requiring a compatible bedtools installation; setup mismatches break the score extraction.
- The result is specific to GM12878/EBV cell-type-specific eQTLs in open chromatin or CTCF sites; generalization to other cell types or peak sets is not established by this analysis.

## Evidence
