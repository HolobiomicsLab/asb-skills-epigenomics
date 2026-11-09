---
name: tad-boundary-prediction-benchmarking
description: Use when comparing TAD/TLD detection algorithms on very sparse Hi-C contact
  matrices (e.g., downsampled Rao et al. bulk data at 0.1%/0.05%/0.025%, 40 kb binsize)
  and ranking them by AMI and WS.
when_to_use_negative:
- The goal is high-level (larger) TLD hierarchy prediction on well-covered bulk data
  rather than benchmarking on ultra-sparse single-cell-level data — deDoc2.w and deDoc2.s
  target higher- vs. lower-level TLDs respectively, and this skill specifically evaluates
  sparse-data robustness
- Input is not a genome-binned Hi-C contact map (e.g., already-imputed single-cell
  embeddings or ChIP-seq peaks) — the skill operates on raw contact matrices treated
  as weighted graphs and requires no normalization/imputation
- Comparing predictors on cell-type classification or embedding quality rather than
  TLD boundary accuracy (AMI/WS against reference TLDs)
edam_operation: http://edamontology.org/operation_3901
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: deDoc2
  role: Primary TLD predictor under benchmark; deDoc2.w globally minimizes 2D structural
    entropy by dynamic programming over the whole contact map, deDoc2.s over 10 Mb
    sliding windows
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Competitor TLD predictor (predecessor using high-dimensional structural entropy)
    run with default settings on downsampled data
  repo: https://github.com/yinxc/structural-information-minimisation
- name: deTOKI
  role: Competitor TLD predictor (NMF-based) run with default settings on downsampled
    data
- name: scHiCluster
  role: Competitor TLD predictor run with default settings on downsampled data
- name: Higashi
  role: Competitor TLD predictor run with nbr = 0, embedding_epoch = 1, no_nbr_epoch
    = 20 on downsampled data
- name: SpectralTAD
  role: Competitor TLD predictor run with default settings on downsampled data
- name: GRiNCH
  role: Competitor TLD predictor run with default settings on downsampled data
- name: IS
  role: Competitor TLD predictor run with default settings on downsampled data
- name: scikit-learn
  role: Provides adjusted_mutual_info_score used to quantify agreement between predicted
    and reference TLD partitions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:fee15dc67c9c9aa690b630ee026ffecd91a16513d1a12c659d10a0cd0c07d31f
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
  edam_operation: http://edamontology.org/operation_3901
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tad-boundary-prediction-benchmarking

## Summary

Benchmark protocol for comparing TAD-like domain (TLD) predictors on ultra-sparse bulk Hi-C data downsampled to single-cell-level sparsity, quantifying prediction accuracy with adjusted mutual information (AMI) and weighted similarity (WS), plus 2D structural entropy and modularity indices.

## When to use

Use when comparing TAD/TLD detection algorithms on very sparse Hi-C contact matrices (e.g., downsampled Rao et al. bulk data at 0.1%/0.05%/0.025%, 40 kb binsize) and ranking them by AMI and WS.

## When NOT to use

- The goal is high-level (larger) TLD hierarchy prediction on well-covered bulk data rather than benchmarking on ultra-sparse single-cell-level data — deDoc2.w and deDoc2.s target higher- vs. lower-level TLDs respectively, and this skill specifically evaluates sparse-data robustness
- Input is not a genome-binned Hi-C contact map (e.g., already-imputed single-cell embeddings or ChIP-seq peaks) — the skill operates on raw contact matrices treated as weighted graphs and requires no normalization/imputation
- Comparing predictors on cell-type classification or embedding quality rather than TLD boundary accuracy (AMI/WS against reference TLDs)

## Inputs

- Bulk Hi-C contact matrix at 40 kb binsize (contact matrix format: n×n tab-split matrix, or sparse format: first line bin count then 'from_bin to_bin edge_weight' lines)
- Downsampled Hi-C matrices at 0.1%, 0.05%, and 0.025% sampling rates from Rao et al.
- Reference TLD partitions predicted from the raw (non-downsampled) data for WS computation

## Outputs

- Per-predictor TLD partitions (deDoc2 .TAD for deDoc2.w and .window.TAD for deDoc2.s; one TLD per line, space-split bin IDs numbering from 1)
- AMI scores (sklearn adjusted_mutual_info_score) comparing predicted vs. reference TLDs at each sampling rate
- Weighted similarity (WS) scores per predictor per sampling rate
- 2D structural entropy and modularity index values per predictor

## How to apply

Downsample bulk Hi-C data (e.g., Rao et al.) to 0.1%, 0.05%, and 0.025% sampling rates at the default 40 kb binsize, and interpret each contact matrix as a weighted graph, adding the self-loop value r = 2m/n0 for isolated vertices as specified in deDoc2.w. Run deDoc2.w (global dynamic-programming minimization of 2D structural entropy, TLD length capped at 10 Mb via -maxTLDsize) and deDoc2.s (10 Mb sliding windows) with default settings, alongside competitor predictors deDoc, IS, deTOKI, scHiCluster, SpectralTAD, and GRiNCH with defaults; run Higashi with nbr = 0, embedding_epoch = 1, and no_nbr_epoch = 20 for the downsampled data. Compute adjusted mutual information between predicted and reference TLDs using sklearn.metrics.adjusted_mutual_info_score, including all predicted TLDs and intermediate windows in T and K. Compute weighted similarity WS by placing TLDs predicted from raw data in T and those from downsampled data in K, excluding intermediate windows from both sets. Compute the 2D structural entropy index by building a height-2 encoding tree from each predictor's TLDs (placing gap-region vertices directly on root λ), and compute modularity after replacing zero diagonal elements and log2-processing the remaining matrix entries.

## Related tools

- **deDoc2** (Primary TLD predictor under benchmark; deDoc2.w globally minimizes 2D structural entropy by dynamic programming over the whole contact map, deDoc2.s over 10 Mb sliding windows) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Competitor TLD predictor (predecessor using high-dimensional structural entropy) run with default settings on downsampled data) — https://github.com/yinxc/structural-information-minimisation
- **deTOKI** (Competitor TLD predictor (NMF-based) run with default settings on downsampled data)
- **scHiCluster** (Competitor TLD predictor run with default settings on downsampled data)
- **Higashi** (Competitor TLD predictor run with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 on downsampled data)
- **SpectralTAD** (Competitor TLD predictor run with default settings on downsampled data)
- **GRiNCH** (Competitor TLD predictor run with default settings on downsampled data)
- **IS** (Competitor TLD predictor run with default settings on downsampled data)
- **scikit-learn** (Provides adjusted_mutual_info_score used to quantify agreement between predicted and reference TLD partitions)

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- deDoc2.w achieves AMI and WS at or above every competitor (deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, Higashi) at each of the 0.1%, 0.05%, and 0.025% sampling rates
- Predicted TLD output files (.TAD, .window.TAD) contain one TLD per line with space-split bin IDs numbering from 1; single-bin lines correspond to non-TAD regions and are excluded
- 2D structural entropy computed on a height-2 encoding tree is low (favorable) and modularity is high relative to competitors, matching the paper's reported ranking
- AMI computation includes all predicted TLDs and intermediate windows in T and K, while WS excludes intermediate windows from both sets — verifying this bookkeeping distinguishes correct metric application
- Higashi parameters for downsampled data are exactly nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20 (all other tools at defaults)

## Limitations

- No version numbers stated for deDoc2 or any compared tool ('All test TLD predictors were run with default settings, except for Higashi'), hindering reproducibility
- No data accession IDs for the downsampled datasets are provided in the context, and supporting information (e.g., Table S4, Figures S2/S3) is not included
- A single bin output in one line should be removed as non-TAD regions; insufficient TLDs (e.g., at the '2n DNA' stage) require excluding those cells/data from analysis

## Evidence

- [results] we generated a series of downsampled data from Rao et al., with sampling rates of 0.1%, 0.05%, and 0.025%: [evidence span withheld — anchor status: not_found]
- [results] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [methods] deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy.: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [readme] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively.: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [methods] All test TLD predictors were run with default settings, except for Higashi: "All test TLD predictors were run with default settings, except for Higashi"
