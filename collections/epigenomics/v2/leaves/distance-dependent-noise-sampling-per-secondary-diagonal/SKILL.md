---
name: distance-dependent-noise-sampling-per-secondary-diagonal
description: 'Use when stress-testing TLD/TAD prediction robustness on sparse (single-cell-like)
  Hi-C matrices: noise must respect distance-dependent decay, so each secondary diagonal
  is sampled independently rather than uniformly across the whole matrix.'
when_to_use_negative:
- The goal is TLD prediction on a single clean matrix rather than noise-robustness
  benchmarking across a perturbation series
- Input is experimental single-cell Hi-C whose sparsity is intrinsic; the perturbation
  would confound natural sparsity with synthetic noise
- The noise model must be uniform over the matrix rather than distance-dependent,
  since this scheme samples each secondary diagonal separately by its contact probability
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: deDoc2
  role: TLD predictor rerun on noisy and original downsampled matrices (deDoc2.w and
    deDoc2.s) to obtain AMI/WS robustness values
  repo: https://github.com/zengguangjie/deDoc2
- name: scikit-learn
  role: Computes adjusted mutual information between original and noisy TLD predictions
    via sklearn.metrics.adjusted_mutual_info_score
- name: deDoc
  role: Prior structural-entropy TLD predictor included as a baseline on the noisy
    matrices
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:505fb4d9e358679a73261a61966eabf568e1df1c5f7e556e47eb108547fb8be2
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Distance-dependent noise sampling per secondary diagonal

## Summary

A controlled perturbation scheme that adds arbitrary noise to downsampled bulk Hi-C contact matrices (0.025% sampling rate, 40 kb binsize) by sampling 10%, 20%, or 40% of contact entries separately within each secondary diagonal according to its distance-dependent contact probability, used to benchmark robustness of TLD predictors such as deDoc2.

## When to use

Use when stress-testing TLD/TAD prediction robustness on sparse (single-cell-like) Hi-C matrices: noise must respect distance-dependent decay, so each secondary diagonal is sampled independently rather than uniformly across the whole matrix.

## When NOT to use

- The goal is TLD prediction on a single clean matrix rather than noise-robustness benchmarking across a perturbation series
- Input is experimental single-cell Hi-C whose sparsity is intrinsic; the perturbation would confound natural sparsity with synthetic noise
- The noise model must be uniform over the matrix rather than distance-dependent, since this scheme samples each secondary diagonal separately by its contact probability

## Inputs

- Downsampled bulk Hi-C intra-chromosome contact matrix at 0.025% sampling rate (Rao et al.), 40 kb binsize, symmetric n x n tab-split or sparse from_bin/to_bin/weight format
- Distance-dependent contact probability per secondary diagonal computed as in Boost-HiC
- Noise rates Pn in {10%, 20%, 40%} of original contact count

## Outputs

- Three noisy contact matrices per tested rate, restricted to rows/columns with at least one nonzero entry
- TLD predictions from each predictor on original and noisy matrices
- Per-predictor AMI and WS values per noise level assembled into noise_robustness_ami_ws.csv

## How to apply

Load the downsampled bulk Hi-C contact matrices (Rao et al., 0.025% sampling rate, default 40 kb binsize). For each noise level Pn (10%, 20%, 40% of the number of contacts in the original downsampled matrix), randomly select that proportion of entries with replacement, sampling each secondary diagonal separately according to its distance-dependent contact probability computed as in Boost-HiC, add a constant of one to each selected entry, and restrict the selection to rows/columns with at least one nonzero entry. This per-diagonal scheme mimics arbitrary data noise while preserving the empirical distance-dependent contact profile, unlike naive uniform perturbation. Rerun TLD predictors (deDoc2, deDoc, SpectralTAD-class network detectors, and score-based detectors such as scHiCluster, deTOKI, Higashi) on the noisy matrices with default settings, then quantify preservation via adjusted mutual information (sklearn.metrics.adjusted_mutual_info_score, including intermediate windows) and weighted similarity (excluding intermediate windows), placing original predictions in T and noisy predictions in K.

## Related tools

- **deDoc2** (TLD predictor rerun on noisy and original downsampled matrices (deDoc2.w and deDoc2.s) to obtain AMI/WS robustness values) — https://github.com/zengguangjie/deDoc2
- **scikit-learn** (Computes adjusted mutual information between original and noisy TLD predictions via sklearn.metrics.adjusted_mutual_info_score)
- **deDoc** (Prior structural-entropy TLD predictor included as a baseline on the noisy matrices) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- noise_robustness_ami_ws.csv exists and contains AMI and WS for each predictor at 10%, 20%, and 40% noise rates
- AMI and WS between noisy and original TLDs decrease only slightly as noise increases, reproducing the reported noise robustness of deDoc2
- deDoc2.w attains the highest AMI and WS among all predictors at each noise level
- Number of perturbed entries equals Pn of the original downsampled matrix contact count, with additions of a constant of one and selection restricted to nonzero rows/columns
- Per-diagonal sampling respects each secondary diagonal's distance-dependent contact probability rather than a uniform whole-matrix distribution

## Limitations

- Noise is synthetic and added with replacement; it mimics arbitrary noise but does not model systematic experimental biases of real single-cell Hi-C
- Tested only at sampling rates of 0.1%, 0.05%, and 0.025% (robustness shown for 0.025%, default 40 kb binsize) and noise rates up to 40%
- Software version numbers of the TLD predictors are not stated in the article

## Evidence

- [methods] Noise was introduced to the Hi‐C matrix as follows.[71]: [evidence span withheld — anchor status: not_found]
- [results] we randomly modified 10%, 20%, and 40% of entries in the Hi‐C matrices with a replacement to mimic data noise: "we randomly modified 10%, 20%, and 40% of entries in the Hi‐C matrices with a replacement to mimic data noise"
- [results] we generated a series of downsampled data from Rao et al., with sampling rates of 0.1%, 0.05%, and 0.025%: [evidence span withheld — anchor status: not_found]
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
- [readme] deDoc2 relys on java environment.: "deDoc2 relys on java environment."
