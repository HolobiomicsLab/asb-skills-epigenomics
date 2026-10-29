---
name: distance-dependent-background-removal
description: Use when a preprocessed 40 kb-resolution cis-interaction Hi-C or capture
  Hi-C contact map still shows distance-decay-dominated ligation frequencies after
  coverage-bias removal and you need background-corrected contact frequencies.
when_to_use_negative:
- The input is raw inter-chromosomal (trans) interaction data — covNorm focuses on
  cis-interactions, and inter-chromosomal interactions were filtered in the article's
  pipeline.
- Only coverage bias (unequal fragment coverage) is present and the distance-decay
  signal itself is the quantity of interest, e.g. building expected-vs-distance curves
  for compartment analysis.
- The contact map has not been processed by the coverage-bias-removing step, since
  the background removal is applied sequentially after it in the covNorm workflow.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: covNorm
  role: R package implementing the negative binomial model-based coverage normalization
    followed by the distance-dependent background removal step
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: HiC-spector
  role: Used to evaluate the effect of background removal via the run_reproducibility_v2.py
    script (get_reproducibility, num_evec = 20) on replicate contact maps
  repo: https://github.com/gersteinlab/HiC-spector
- name: BEDTools coverage
  role: Produces the 40 kb-resolution per-fragment coverage profiles used to build
    the covNorm input fed into background removal
  repo: https://github.com/arq5x/bedtools2
- name: iced
  role: Baseline comparison package running ICE and SCN normalization against which
    the covNorm background-removal pipeline was benchmarked
  repo: https://github.com/hiclib/iced
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:ed62392bf389a2e5fb4c0fca8a85c12aaa81572d506e2c5ed6df30b8c3f94a1c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kaistcbfg/covNormRpkg
  repo_url: https://github.com/kaistcbfg/covNormRpkg
  tools:
  - BEDTools coverage
  - HiC-spector
  - HiCcompare
  - R
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# distance-dependent-background-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A covNorm normalization step that removes the genomic distance-dependent background signal from Hi-C contact maps after negative binomial coverage-bias removal, applied sequentially to improve contact map reproducibility.

## When to use

Use when a preprocessed 40 kb-resolution cis-interaction Hi-C or capture Hi-C contact map still shows distance-decay-dominated ligation frequencies after coverage-bias removal and you need background-corrected contact frequencies.

## When NOT to use

- The input is raw inter-chromosomal (trans) interaction data — covNorm focuses on cis-interactions, and inter-chromosomal interactions were filtered in the article's pipeline.
- Only coverage bias (unequal fragment coverage) is present and the distance-decay signal itself is the quantity of interest, e.g. building expected-vs-distance curves for compartment analysis.
- The contact map has not been processed by the coverage-bias-removing step, since the background removal is applied sequentially after it in the covNorm workflow.

## Inputs

- Coverage-bias-corrected Hi-C contact map in covNorm input format (frag1, frag2, cov_frag1, cov_frag2, freq, dist)
- 40 kb-resolution chr17 cis-interaction maps from GM19204 and GM19240 in situ Hi-C replicates
- Distance-dependent average contact-frequency profile (expected R_ij vs genomic distance)

## Outputs

- Background-corrected normalized contact map (coverage-bias and distance-dependent background removed)
- Significance scores of chromatin contacts (optional sequential step)

## How to apply

After the negative binomial GLM bias-removing step (log(u_ij) = b0 + b1*Ci + b2*Cj fitted via MASS::glm.nb on the covNorm input columns frag1, frag2, cov_frag1, cov_frag2, freq, dist), apply the distance-dependent background normalization step on the same fragment pairs. Compute the average contact frequency as a function of genomic distance (avg(R_ij)) and remove this expected distance-dependent background from the observed ligation frequencies, using a pseudocount to stabilize low-count bins. In the benchmark, the bias-removing and background steps were applied sequentially to GM19204 and GM19240 chr17 cis-interaction maps at 40 kb resolution before HiC-spector reproducibility scoring with num_evec = 20. Significant interactions can also be called after background removal, and the same pipeline extends to derivative protocols such as pcHi-C with promoter-side coverage threshold 200 and other-side threshold 50.

## Related tools

- **covNorm** (R package implementing the negative binomial model-based coverage normalization followed by the distance-dependent background removal step) — https://github.com/kaistcbfg/covNormRpkg
- **HiC-spector** (Used to evaluate the effect of background removal via the run_reproducibility_v2.py script (get_reproducibility, num_evec = 20) on replicate contact maps) — https://github.com/gersteinlab/HiC-spector
- **BEDTools coverage** (Produces the 40 kb-resolution per-fragment coverage profiles used to build the covNorm input fed into background removal) — https://github.com/arq5x/bedtools2
- **iced** (Baseline comparison package running ICE and SCN normalization against which the covNorm background-removal pipeline was benchmarked) — https://github.com/hiclib/iced

## Examples

```
Rscript -e 'library(covNorm); norm_map <- covNorm_remove_background(input_map, cov_cutoff = 200, resolution = 40000)'
```

## Evaluation signals

- After background removal, the replicate (GM19204 vs GM19240) HiC-spector score (num_evec = 20) should improve over raw data — the article reports a significant paired t-test change for covNorm vs raw (p-value = 0.019) and the highest median score among methods.
- The corrected map should no longer be dominated by distance decay: average contact frequency vs genomic distance (expected R_ij) should be approximately flat after removal.
- Compartment A/B profile reproducibility between replicates should show correlation coefficients at or above those of ICE/KR/SCN baselines (covNorm recorded the best Pearson correlation and Spearman's rho in the article).
- Validation of the output CSV (hic_spector_median_scores.csv) should confirm all five methods (covNorm, ICE, KR, SCN, raw) are scored and covNorm's median is highest or comparable-best, with paired t-test p-values > 0.05 between methods.
- Preceding filter steps must be verified: zero-valued fragment pairs removed, fragments with coverage below 200 at 40 kb resolution filtered, and 50% of coverage values shuffled.

## Limitations

- The default coverage cut-off threshold of 200 at 40 kb resolution is dataset-dependent and should be adjusted by the user for other resolutions or protocols.
- A pseudocount (avg(R_ij)) must be added to avoid instability from near-zero expected frequencies at short or long distances.
- The article does not state an explicit limitation section for covNorm, and no version number or changelog is available for the software repository.
- Performance differences between methods were not statistically significant (paired t-test, p-values > 0.05), so the improvement from background removal is modest relative to baseline normalizations.

## Evidence
