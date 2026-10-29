---
name: weighted-similarity-scoring
description: Use when comparing TLD/TAD partitions predicted from a raw bulk Hi-C
  matrix against partitions from downsampled (e.g. 0.1%, 0.05%, 0.025%) or otherwise
  degraded Hi-C data to quantify predictor robustness to sparsity.
when_to_use_negative:
- The comparison target is internal clustering quality of a single partition rather
  than cross-condition (raw vs. downsampled) partition agreement — use structural
  entropy/modularity indices or AMI alone instead
- Data are already single-cell scHi-C experiments rather than downsampled bulk matrices
  — the WS benchmark in the paper is defined for raw-vs-downsampled bulk Hi-C pairs
- Predicted TLDs include intermediate windows that cannot be removed — including them
  in T and K violates the metric's definition as described in the paper's workflow
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: deDoc2
  role: Primary TLD predictor whose raw-vs-downsampled partitions are scored by WS;
    deDoc2.w for higher-level, deDoc2.s for lower-level TLDs
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline bulk Hi-C TAD predictor scored by WS for comparison
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scikit-learn
  role: Provides adjusted_mutual_info_score used alongside WS to quantify prediction
    accuracy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:6dde5b3943afb53973b68363abca8134ab7c664e742d65c6f9785d10ccae803c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - GRiNCH
  - Higashi
  - IS
  - SpectralTAD
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# weighted-similarity-scoring

## Summary

Benchmark metric that scores the agreement between TAD-like domain (TLD) partitions predicted from raw (reference) Hi-C data and partitions predicted from downsampled/imputed data at single-cell-level sparsity, as used to rank deDoc2.w against deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, and Higashi in the deDoc2 paper.

## When to use

Use when comparing TLD/TAD partitions predicted from a raw bulk Hi-C matrix against partitions from downsampled (e.g. 0.1%, 0.05%, 0.025%) or otherwise degraded Hi-C data to quantify predictor robustness to sparsity.

## When NOT to use

- The comparison target is internal clustering quality of a single partition rather than cross-condition (raw vs. downsampled) partition agreement — use structural entropy/modularity indices or AMI alone instead
- Data are already single-cell scHi-C experiments rather than downsampled bulk matrices — the WS benchmark in the paper is defined for raw-vs-downsampled bulk Hi-C pairs
- Predicted TLDs include intermediate windows that cannot be removed — including them in T and K violates the metric's definition as described in the paper's workflow

## Inputs

- Reference TLD partition (set T) predicted from raw bulk Hi-C contact matrices (Rao et al., binsize 40 kb)
- Test TLD partition (set K) predicted from downsampled bulk Hi-C matrices (0.1%, 0.05%, 0.025% sampling rates)
- Predictor outputs from deDoc2.w, deDoc2.s, deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, and Higashi (Higashi run with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20)

## Outputs

- Weighted similarity (WS) score per predictor per sampling rate
- Adjusted mutual information (AMI) values (sklearn.metrics.adjusted_mutual_info_score) for the same predictor/sampling-rate grid
- Ranking table establishing whether deDoc2.w attains the highest AMI and WS at every sampling rate

## How to apply

Run each TLD predictor (deDoc2.w/deDoc2.s and competitors) with default settings on both the raw Rao et al. bulk Hi-C contact matrices and the downsampled versions at sampling rates 0.1%, 0.05%, and 0.025% with binsize 40 kb. Construct the reference set T from predicted TLDs on the raw data and the test set K from predicted TLDs on the downsampled data, explicitly excluding intermediate windows from both sets. The weighted similarity (WS) then quantifies how faithfully a predictor's domains are recovered under severe sparsity; it is reported alongside adjusted mutual information (AMI, via sklearn.metrics.adjusted_mutual_info_score) computed over all predicted TLDs including intermediate windows in T and K. A predictor is judged superior when its WS and AMI are at or above every competitor at each sampling rate, corroborated by favorable 2D structural entropy (low) and modularity (high) index values. In this benchmark deDoc2.w achieved the highest AMI and WS among all tested algorithms at every sampling rate, confirming first-class performance.

## Related tools

- **deDoc2** (Primary TLD predictor whose raw-vs-downsampled partitions are scored by WS; deDoc2.w for higher-level, deDoc2.s for lower-level TLDs) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline bulk Hi-C TAD predictor scored by WS for comparison) — https://github.com/yinxc/structural-information-minimisation
- **scikit-learn** (Provides adjusted_mutual_info_score used alongside WS to quantify prediction accuracy)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(T, K)
```

## Evaluation signals

- T and K are built from TLDs predicted on raw and downsampled data respectively, with intermediate windows excluded from both sets
- deDoc2.w should attain WS and AMI values at or above every competitor (deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, Higashi) at each of the 0.1%, 0.05%, and 0.025% sampling rates
- Competitors are run with default settings (except Higashi: nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20) so the WS comparison is parameter-controlled
- WS ranking is corroborated by the 2D structural entropy index (height-2 encoding tree, gap-region vertices on root lambda) and modularity after log2-processing the contact matrix

## Limitations

- WS requires paired predictions on raw and downsampled data; no absolute quality score is produced for a single partition
- All test TLD predictors were run with default settings, so tool-specific version numbers and parameter sensitivity are unreported
- Supporting Information tables/figures with the underlying quantitative WS values are not included in the main document

## Evidence

- [results] sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb: "sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb"
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
- [discussion] deDoc2 exhibits first‐class performance on the metrics we assessed in the downsampled, simulated, and experimental scHi‐C data: "deDoc2 exhibits first‐class performance on the metrics we assessed in the downsampled, simulated, and experimental scHi‐C data"
- [readme] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively.: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
