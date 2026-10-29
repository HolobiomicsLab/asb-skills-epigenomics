---
name: bootstrap-confidence-interval-estimation
description: Use when reporting a point estimate of entropy decrease (or similar per-cluster
  statistic) after DGW warping and you need an uncertainty range, e.g. mean 1.72%
  TSS decrease with a 95% bootstrap CI of 0.83–2.81%.
when_to_use_negative:
- Data has not been clustered and warped by DGW yet — there are no warped vs raw distributions
  to compare entropies over
- The statistic of interest is not expressible as a per-cluster (or resamplable) quantity,
  e.g. a single global value with no sampling units to bootstrap
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: DGW
  role: Produces the cluster assignments and warping paths whose entropy decrease
    is being bootstrapped
  repo: https://github.com/lukauskas/dgw
- name: Python
  role: Language in which histograms, Shannon entropies, and the bootstrap resampling/percentile
    computation are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:97758f94a78a663a4cc08c70cb471af8ec79dcf743cc2708b0e4fd6a04e9472b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bootstrap-confidence-interval-estimation

## Summary

Quantifies uncertainty in DGW alignment quality metrics by bootstrapping the entropy decrease of TSS and FSS location distributions over clusters, yielding 95% confidence intervals for mean percent entropy reduction.

## When to use

Use when reporting a point estimate of entropy decrease (or similar per-cluster statistic) after DGW warping and you need an uncertainty range, e.g. mean 1.72% TSS decrease with a 95% bootstrap CI of 0.83–2.81%.

## When NOT to use

- Data has not been clustered and warped by DGW yet — there are no warped vs raw distributions to compare entropies over
- The statistic of interest is not expressible as a per-cluster (or resamplable) quantity, e.g. a single global value with no sampling units to bootstrap

## Inputs

- Per-cluster raw and warped POI position lists or histograms for TSS and FSS (from dgw-extract-gene-regions and DGW warping paths)
- DGW clustering outputs ({prefix}_config.dgw, warping paths, cluster BED files)
- Number of bootstrap replicates and percentile bounds (e.g. 95% CI)

## Outputs

- Mean percent entropy decrease across clusters for TSS and FSS
- 95% bootstrap confidence intervals (e.g. TSS: 0.83–2.81%, FSS: 1.79–3.63%)
- Per-cluster point estimates (e.g. 12.91% TSS, 7.72% FSS in the selected cluster)

## How to apply

After recovering raw and warped POI positions for every DGW cluster, build binned histograms of TSS and FSS positions for the raw data (rescaled to a common length) and the warped data per cluster, and compute the Shannon entropy of each histogram to obtain a percent entropy decrease per cluster. Then resample clusters (or the underlying POI observations) with replacement, recompute the mean percent entropy decrease for each bootstrap replicate, and take the 2.5th and 97.5th percentiles of the replicate distribution as the 95% confidence interval. In the DGW ENCODE analysis this produced a mean TSS entropy decrease of 1.72% (95% Bootstrap CI 0.83–2.81%) and a mean FSS decrease of 2.65% (95% Bootstrap CI 1.79–3.63%), alongside the selected-cluster point estimates of 12.91% (TSS) and 7.72% (FSS). The rationale is that per-cluster decreases are noisy, so only a resampling-based interval conveys whether the warping consistently tightens landmark alignment.

## Related tools

- **DGW** (Produces the cluster assignments and warping paths whose entropy decrease is being bootstrapped) — https://github.com/lukauskas/dgw
- **Python** (Language in which histograms, Shannon entropies, and the bootstrap resampling/percentile computation are implemented)

## Examples

```
import numpy as np; boots = [mean_entropy_decrease(resample(clusters)) for _ in range(10000)]; ci = np.percentile(boots, [2.5, 97.5])
```

## Evaluation signals

- Per-cluster point estimates match reported values: 12.91% decrease for TSS and 7.72% for FSS in the selected cluster
- Mean-across-clusters estimates fall within reported intervals: TSS 1.72% (CI 0.83–2.81%), FSS 2.65% (CI 1.79–3.63%)
- Each bootstrap CI is computed from the replicate distribution's 2.5th and 97.5th percentiles and brackets its own point estimate
- Raw and warped histograms are built over comparable bins (raw rescaled to a common length), so entropy decreases are non-negative in expectation

## Limitations

- Standard hierarchical clustering only succeeds for small noise/modifications, so cluster quality upstream affects the stability of the bootstrapped entropy decreases
- Bootstrapped averages over clusters can be much smaller (1.72–2.65%) than the selected-cluster values (12.91%/7.72%), so the CI must not be conflated with single-cluster effects
- Dendrogram cutting level is a difficult choice and influences cluster membership, hence the bootstrap distribution

## Evidence

- [methods] selected-cluster point estimates: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [abstract] abstract claim of automatic landmark alignment: "DGW automatically recognises and aligns important genomic features such as transcription start sites and splicing sites from histone marks."
- [abstract] tool availability for implementation: "DGW is available as an open source Python package."
