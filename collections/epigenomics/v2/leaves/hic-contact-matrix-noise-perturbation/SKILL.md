---
name: hic-contact-matrix-noise-perturbation
description: Use when benchmarking TAD/TLD predictors for robustness on sparse Hi-C
  data, e.g., single-cell-level downsampled bulk Hi-C matrices (0.025% sampling rate,
  40 kb binsize), by comparing predictions on noisy vs. original matrices.
when_to_use_negative:
- When the input is experimental single-cell Hi-C data whose noise characteristics
  you want to measure rather than simulate — this skill only evaluates predictor robustness
  to synthetic noise
- When the goal is absolute accuracy of TLD boundary positions against an annotated
  ground truth (e.g., ChIP-seq boundary validation) rather than stability under perturbation
- When matrices are so sparse that few or no rows/columns have at least one nonzero
  entry, making the diagonal-wise noise sampling restriction ill-defined
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: TLD predictor whose noisy-vs-original AMI and WS are benchmarked; deDoc2.w
    achieves the highest AMI and WS among all predictors
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Baseline structural-entropy TAD predictor run on noisy matrices for comparison
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scikit-learn
  role: Provides sklearn.metrics.adjusted_mutual_info_score for computing AMI between
    noisy and original TLD predictions
- name: scHiCluster
  role: Score-based imputation-based TLD predictor rerun on noisy matrices as a comparison
    method
- name: deTOKI
  role: NMF-based TLD predictor rerun on noisy matrices as a comparison method
- name: Higashi
  role: TLD predictor rerun on noisy matrices with non-default settings (nbr = 0,
    embedding_epoch = 1, no_nbr_epoch = 20)
- name: SpectralTAD
  role: Network-detector class TLD predictor rerun on noisy matrices as a comparison
    method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:1c8b3a3858f3c216db8332bdd62e58607e8608b251287d3ae718f3b218d861a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - Higashi
  - SpectralTAD
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Hi-C contact matrix noise perturbation

## Summary

A robustness benchmarking protocol that injects controlled noise (10%, 20%, 40% of contacts, sampled per secondary diagonal from distance-dependent contact probabilities) into downsampled Hi-C contact matrices and quantifies preservation of TLD predictions via adjusted mutual information (AMI) and weighted similarity (WS).

## When to use

Use when benchmarking TAD/TLD predictors for robustness on sparse Hi-C data, e.g., single-cell-level downsampled bulk Hi-C matrices (0.025% sampling rate, 40 kb binsize), by comparing predictions on noisy vs. original matrices.

## When NOT to use

- When the input is experimental single-cell Hi-C data whose noise characteristics you want to measure rather than simulate — this skill only evaluates predictor robustness to synthetic noise
- When the goal is absolute accuracy of TLD boundary positions against an annotated ground truth (e.g., ChIP-seq boundary validation) rather than stability under perturbation
- When matrices are so sparse that few or no rows/columns have at least one nonzero entry, making the diagonal-wise noise sampling restriction ill-defined

## Inputs

- Downsampled bulk Hi-C intra-chromosome contact matrices (e.g., Rao et al. at 0.025% sampling rate, 40 kb binsize), in contact matrix or sparse format (first line = number of bins, then from_bin to_bin edge_weight lines)
- Noise rate levels Pn = 10%, 20%, 40% of the number of contacts in the original downsampled matrices
- Original (noise-free) TLD predictions from each predictor as the reference set T

## Outputs

- Noisy Hi-C contact matrices at each noise level (10%, 20%, 40%)
- TLD predictions from each predictor on the noisy matrices
- Table of AMI and WS values per noise level and predictor, saved as noise_robustness_ami_ws.csv

## How to apply

Start from downsampled bulk Hi-C contact matrices (Rao et al., 0.025% sampling rate, default 40 kb binsize). Introduce noise by randomly selecting a proportion Pn of entries with replacement (Pn = 10%, 20%, and 40% of the number of contacts), adding a constant of one; sample each secondary diagonal separately according to its distance-dependent contact probability computed as in Boost-HiC, restricting selection to rows/columns with at least one nonzero entry. Rerun TLD predictors (deDoc, deDoc2, SpectralTAD-class network detectors, and score-based detectors such as scHiCluster, deTOKI, Higashi) with default settings — except Higashi with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 for downsampled data. Compute AMI of noisy vs. original TLD predictions with sklearn.metrics.adjusted_mutual_info_score (including intermediate windows in T and K) and weighted similarity (WS), placing original predictions in T, noisy in K, excluding intermediate windows. The rationale is that a robust predictor's AMI and WS should decrease only slightly as noise rate increases.

## Related tools

- **deDoc2** (TLD predictor whose noisy-vs-original AMI and WS are benchmarked; deDoc2.w achieves the highest AMI and WS among all predictors) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Baseline structural-entropy TAD predictor run on noisy matrices for comparison) — https://github.com/yinxc/structural-information-minimisation
- **scikit-learn** (Provides sklearn.metrics.adjusted_mutual_info_score for computing AMI between noisy and original TLD predictions)
- **scHiCluster** (Score-based imputation-based TLD predictor rerun on noisy matrices as a comparison method)
- **deTOKI** (NMF-based TLD predictor rerun on noisy matrices as a comparison method)
- **Higashi** (TLD predictor rerun on noisy matrices with non-default settings (nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20))
- **SpectralTAD** (Network-detector class TLD predictor rerun on noisy matrices as a comparison method)

## Examples

```
java -jar deDoc2.jar -inputfile ./noisy_20pct_chr18.matrix -binsize 40 -outputfile ./noisy_20pct_chr18
```

## Evaluation signals

- noise_robustness_ami_ws.csv exists and contains AMI and WS values for each predictor at all three noise rates (10%, 20%, 40%)
- AMI and WS of TLDs between noisy and original downsampled data decrease only slightly as the noise rate increases
- deDoc2.w achieves the highest AMI and WS among all predictors at every noise level
- Noisy matrices are symmetric (input Hi-C matrix should be symmetric) and noise was only added to rows/columns with at least one nonzero entry
- All predictors were run with default settings, except Higashi (nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20)

## Limitations

- Noise model assumes entries are modified with replacement and a constant of one is added, following the Boost-HiC distance-dependent sampling scheme; other noise models (deletions, scaling) are not covered
- The benchmark is anchored on downsampled bulk Hi-C (0.025% sampling rate); conclusions may not transfer to other sparsity levels or experimental single-cell data
- WS excludes intermediate windows of domains, so hierarchy-level agreement (nested subdomains) is not scored by WS, only by AMI which includes intermediate windows
- Software version numbers of compared tools are not stated in the source document

## Evidence

- [methods] Noise was introduced to the Hi-C matrix as follows.[71]: [evidence span withheld — anchor status: not_found]
- [results] we randomly modified 10%, 20%, and 40% of entries in the Hi-C matrices with a replacement to mimic data noise: "we randomly modified 10%, 20%, and 40% of entries in the Hi‐C matrices with a replacement to mimic data noise"
- [results] sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb: "sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb"
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
- [readme] deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable.: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi‐C data is suitable."
