---
name: tld-prediction-robustness-benchmarking
description: Use when evaluating whether a TLD predictor (e.g., deDoc2, scHiCluster,
  deTOKI, Higashi) preserves its domain calls when Hi-C contact matrices are perturbed
  by controlled noise levels such as 10%, 20%, or 40%.
when_to_use_negative:
- When the goal is absolute TLD prediction accuracy on ground-truth data rather than
  stability under perturbation — this skill compares noisy vs. original predictions,
  not predictions vs. known annotations
- When input matrices are already heavily processed or imputed (e.g., RWR-imputed),
  since deDoc2 is designed to decode TLDs from raw, unimputed single-cell Hi-C data
  and noise modeling assumes sparse downsampled inputs
- When binsize must be determined first — use the NDI-based binsize determination
  (deDoc2 -NDI flag) instead, as this skill assumes a fixed default 40 kb binsize
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Primary TLD predictor under test; run via deDoc2.jar on original and noisy
    matrices; deDoc2.w achieved the highest AMI and WS among all predictors
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline structural-entropy TAD predictor included in the comparison set on
    noisy matrices
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scHiCluster
  role: Score-based imputation-dependent TLD predictor rerun on noisy matrices for
    comparison
- name: deTOKI
  role: NMF-based single-cell TLD predictor rerun on noisy matrices for comparison
- name: Higashi
  role: Graph-based predictor run with nbr = 0, embedding_epoch = 1, no_nbr_epoch
    = 20 for downsampled data in the comparison
- name: scikit-learn
  role: Provides adjusted_mutual_info_score for computing AMI between original and
    noisy TLD predictions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:748df924b30bb9dcbc0f7285acb9d4c07ef8b40ab99eebda4cc7aed3bc348e33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - Higashi
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# TLD prediction robustness benchmarking

## Summary

A benchmarking protocol for assessing how robust TAD-like domain (TLD) predictors are to arbitrary data noise, by adding controlled noise to downsampled Hi-C matrices and comparing TLD predictions via adjusted mutual information (AMI) and weighted similarity (WS).

## When to use

Use when evaluating whether a TLD predictor (e.g., deDoc2, scHiCluster, deTOKI, Higashi) preserves its domain calls when Hi-C contact matrices are perturbed by controlled noise levels such as 10%, 20%, or 40%.

## When NOT to use

- When the goal is absolute TLD prediction accuracy on ground-truth data rather than stability under perturbation — this skill compares noisy vs. original predictions, not predictions vs. known annotations
- When input matrices are already heavily processed or imputed (e.g., RWR-imputed), since deDoc2 is designed to decode TLDs from raw, unimputed single-cell Hi-C data and noise modeling assumes sparse downsampled inputs
- When binsize must be determined first — use the NDI-based binsize determination (deDoc2 -NDI flag) instead, as this skill assumes a fixed default 40 kb binsize

## Inputs

- Downsampled bulk Hi-C contact matrices (e.g., Rao et al., 0.025% sampling rate, 40 kb binsize)
- Contact matrix format (n x n intra-chromosome tab-split matrix) or sparse format (first line = number of bins, then from_bin to_bin edge_weight lines) as accepted by deDoc2
- Noise rate specifications (Pn = 10%, 20%, 40% of the number of contacts in the original downsampled matrices)

## Outputs

- TLD predictions from each predictor on original and noisy matrices (deDoc2 outputs .TAD and .window.TAD partition files, node ids space-split from 1)
- AMI and WS values per noise level and predictor, assembled into a table such as noise_robustness_ami_ws.csv

## How to apply

Start with downsampled bulk Hi-C contact matrices (e.g., Rao et al. data at 0.025% sampling rate, default 40 kb binsize). Introduce noise by randomly selecting a proportion Pn (10%, 20%, 40% of contacts) of entries with replacement, adding a constant of one, sampling each secondary diagonal separately according to its distance-dependent contact probability as in Boost-HiC, and restricting selection to rows/columns with at least one nonzero entry. Rerun the TLD predictors (deDoc, deDoc2.w/deDoc2.s, SpectralTAD-class network detectors, and score-based detectors such as scHiCluster, deTOKI, Higashi) on both original and noisy matrices with default settings (Higashi exception: nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 for downsampled data). Compute AMI with sklearn.metrics.adjusted_mutual_info_score, including intermediate windows of TLDs in T and K, and WS with original predictions in T and noisy predictions in K, excluding intermediate windows of domains. A predictor is judged robust if AMI and WS decrease only slightly as noise rate increases; in the source study, deDoc2.w achieved the highest AMI and WS among all predictors. Assemble per-noise-level, per-predictor values into a table such as noise_robustness_ami_ws.csv.

## Related tools

- **deDoc2** (Primary TLD predictor under test; run via deDoc2.jar on original and noisy matrices; deDoc2.w achieved the highest AMI and WS among all predictors) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline structural-entropy TAD predictor included in the comparison set on noisy matrices) — https://github.com/yinxc/structural-information-minimisation
- **scHiCluster** (Score-based imputation-dependent TLD predictor rerun on noisy matrices for comparison)
- **deTOKI** (NMF-based single-cell TLD predictor rerun on noisy matrices for comparison)
- **Higashi** (Graph-based predictor run with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 for downsampled data in the comparison)
- **scikit-learn** (Provides adjusted_mutual_info_score for computing AMI between original and noisy TLD predictions)

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- noise_robustness_ami_ws.csv exists and contains AMI and WS values for every predictor at 10%, 20%, and 40% noise rates
- AMI (computed with sklearn.metrics.adjusted_mutual_info_score, including intermediate windows in T and K) decreases only slightly as noise rate increases for a robust predictor
- WS computed with original predictions in T and noisy predictions in K, excluding intermediate domains, remains high across noise levels; deDoc2.w expected to attain the highest AMI and WS
- deDoc2 output partitions parse as TLDs with node ids numbering from 1, and single-bin lines are removed as non-TAD regions before scoring
- Noisy matrices are valid Hi-C graphs: entries modified with replacement plus a constant of one, diagonal-aware sampling, and only rows/columns with at least one nonzero entry touched

## Limitations

- All predictors were run with default settings (except Higashi-specific parameters), so results reflect defaults rather than tuned configurations
- SuperTAD requires several orders of magnitude higher CPU time and memory space to solve high-dimensional structural entropy, limiting its practical inclusion
- Noise model is specific: random modification with replacement adding a constant of one with diagonal-aware sampling; other noise regimes are not covered
- No software version numbers stated for deDoc2 or any compared tool in the context

## Evidence

- [results] deDoc2 is robust to noise because both AMI and WS of TLDs between noisy and original downsampled data decreased only slightly when noise rate increased, with deDoc2.w achieving the highest AMI and WS among all predictors.: "deDoc2 is robust to noise because both AMI and WS of TLDs between noisy and original downsampled data decreased only slightly when noise rate increased"
- [methods] Noise was introduced to the Hi-C matrix as follows.: [evidence span withheld — anchor status: not_found]
- [results] Noise introduced by randomly modifying 10%, 20%, and 40% of entries in Hi-C matrices with replacement to mimic data noise.: "we randomly modified 10%, 20%, and 40% of entries in the Hi‐C matrices with a replacement to mimic data noise"
- [readme] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively.: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
