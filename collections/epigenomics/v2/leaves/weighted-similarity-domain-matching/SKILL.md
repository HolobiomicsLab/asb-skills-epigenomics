---
name: weighted-similarity-domain-matching
description: Use when two sets of TAD/TLD boundary predictions must be compared quantitatively
  (e.g., noisy vs. reference Hi-C calls, or two predictors on the same matrix) and
  a contact-weighted domain-overlap score, excluding intermediate windows, is needed.
when_to_use_negative:
- When the question is preserving hierarchy including intermediate/nested windows
  — WS explicitly excludes intermediate windows of domains; use AMI including T and
  K intermediate windows instead
- When inputs are raw contact matrices rather than already-predicted TLD boundary
  partitions — predictors (deDoc2, scHiCluster, deTOKI, Higashi, etc.) must run first
- When inputs are bulk-depth Hi-C data where domain calls are already stable, so noise-robustness
  fidelity scoring is not the analysis goal
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Generates the TLD partitions (deDoc2.w for higher-level, deDoc2.s for lower-level)
    whose noisy-vs-original overlap is scored with WS; achieved the highest AMI and
    WS among all predictors
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline bulk Hi-C TLD predictor re-run on noisy matrices for WS/AMI comparison
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scikit-learn
  role: Supplies adjusted_mutual_info_score used as the companion fidelity metric
    to WS
- name: scHiCluster
  role: Imputation-based TLD predictor re-run on noisy matrices as a comparison point
    in the WS/AMI benchmark
- name: deTOKI
  role: NMF-based TLD predictor re-run on noisy matrices as a comparison point in
    the WS/AMI benchmark
- name: Higashi
  role: TLD predictor re-run on noisy matrices with nbr = 0, embedding_epoch = 1,
    no_nbr_epoch = 20 for downsampled data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:3de004ad2a97fd55535b8e40a96328ab1cde9ca3df1f0e1646f9e7560b2253ca
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Weighted similarity (WS) domain-matching metric

## Summary

Computes the weighted similarity (WS) between two sets of predicted TAD-like domains (TLDs) — e.g., TLDs called on noisy versus original downsampled Hi-C matrices — as the fidelity metric used alongside adjusted mutual information (AMI) to benchmark TLD predictors such as deDoc2, deDoc, scHiCluster, deTOKI, Higashi, and SpectralTAD.

## When to use

Use when two sets of TAD/TLD boundary predictions must be compared quantitatively (e.g., noisy vs. reference Hi-C calls, or two predictors on the same matrix) and a contact-weighted domain-overlap score, excluding intermediate windows, is needed.

## When NOT to use

- When the question is preserving hierarchy including intermediate/nested windows — WS explicitly excludes intermediate windows of domains; use AMI including T and K intermediate windows instead
- When inputs are raw contact matrices rather than already-predicted TLD boundary partitions — predictors (deDoc2, scHiCluster, deTOKI, Higashi, etc.) must run first
- When inputs are bulk-depth Hi-C data where domain calls are already stable, so noise-robustness fidelity scoring is not the analysis goal

## Inputs

- TLD predictions on the reference (original) Hi-C matrix, set T (e.g., .TAD / .window.TAD partition files from deDoc2 on 0.025%-downsampled Rao et al. data, 40 kb binsize)
- TLD predictions on the perturbed (noisy) Hi-C matrix, set K (after adding 10%, 20%, or 40% matrix noise)
- Hi-C contact matrices at single-cell-level sparsity (sampling rates 0.1%, 0.05%, 0.025%) in contact-matrix or sparse edge-list format

## Outputs

- Per-predictor, per-noise-level WS values (noise_robustness_ami_ws.csv table)
- Paired AMI values from sklearn.metrics.adjusted_mutual_info_score for the same predictor/noise combinations

## How to apply

Run each TLD predictor (deDoc, deDoc2.w, deDoc2.s, scHiCluster, deTOKI, Higashi, SpectralTAD-class network/score detectors) on both the reference and perturbed Hi-C contact matrices with default settings — except Higashi on downsampled data, which uses nbr = 0, embedding_epoch = 1, and no_nbr_epoch = 20. Treat reference predictions as set T and perturbed (noisy) predictions as set K. Compute the weighted similarity (WS) by matching domains between T and K weighted by contact counts, explicitly excluding intermediate windows of domains so only resolved TLDs contribute to the score. Report WS alongside AMI computed with sklearn.metrics.adjusted_mutual_info_score (which, unlike WS, includes intermediate windows in T and K). Interpret higher WS/AMI as better robustness: in the deDoc2 study, AMI and WS of TLDs between noisy and original 0.025%-downsampled Rao et al. matrices decreased only slightly as the added-noise rate rose from 10% to 20% to 40%, with deDoc2.w achieving the highest AMI and WS among all predictors.

## Related tools

- **deDoc2** (Generates the TLD partitions (deDoc2.w for higher-level, deDoc2.s for lower-level) whose noisy-vs-original overlap is scored with WS; achieved the highest AMI and WS among all predictors) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline bulk Hi-C TLD predictor re-run on noisy matrices for WS/AMI comparison) — https://github.com/yinxc/structural-information-minimisation
- **scikit-learn** (Supplies adjusted_mutual_info_score used as the companion fidelity metric to WS)
- **scHiCluster** (Imputation-based TLD predictor re-run on noisy matrices as a comparison point in the WS/AMI benchmark)
- **deTOKI** (NMF-based TLD predictor re-run on noisy matrices as a comparison point in the WS/AMI benchmark)
- **Higashi** (TLD predictor re-run on noisy matrices with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 for downsampled data)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(labels_original, labels_noisy)
```

## Evaluation signals

- noise_robustness_ami_ws.csv exists and contains AMI and WS rows for each predictor at 10%, 20%, and 40% noise rates
- WS and AMI values decrease only slightly as the noise rate increases from 10% to 40%, reproducing the reported noise-robustness pattern
- deDoc2.w yields the highest AMI and WS among all compared predictors in the assembled table
- WS computation excludes intermediate windows of domains, while AMI (sklearn adjusted_mutual_info_score) includes them — check the two metrics are not conflated
- Predictors run with default settings except Higashi on downsampled data (nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20)

## Limitations

- WS excludes intermediate windows of domains, so changes in hierarchical/nested structure are not captured by this metric alone
- Robustness was assessed at single-cell-level sparsity (0.025% sampling rate) with a specific noise model (randomly adding one to a proportion Pn of entries, second diagonals sampled by distance-dependent contact probability as in Boost-HiC, restricted to rows/columns with at least one nonzero entry); other noise models may behave differently
- Original predictions are placed in T and noisy predictions in K — the metric direction/orientation must match the paper's convention for comparability
- The discussion section notes no version numbers for deDoc2 or compared tools, so exact-version reproducibility is limited

## Evidence

- [results] Noise introduced by randomly modifying 10%, 20%, and 40% of entries in Hi-C matrices with replacement: "we randomly modified 10%, 20%, and 40% of entries in the Hi‐C matrices with a replacement to mimic data noise"
- [results] Downsampled benchmark data at single-cell-level sparsity: [evidence span withheld — anchor status: not_found]
- [results] deDoc2.w and deDoc2.s produce the two levels of TLD predictions being matched: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [methods] Predictor run settings for the noisy-matrix benchmark: "All test TLD predictors were run with default settings, except for Higashi"
- [readme] deDoc2 command-line invocation producing the .TAD partitions compared by WS: "java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40"
