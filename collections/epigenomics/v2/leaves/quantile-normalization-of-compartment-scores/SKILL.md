---
name: quantile-normalization-of-compartment-scores
description: Use when compartment scores (PC1/PC2 vectors) have been computed for
  multiple Hi-C datasets or replicates and must be made comparable across conditions
  prior to differential compartment analysis.
when_to_use_negative:
- Only a single Hi-C contact map is being analyzed and no cross-condition comparison
  of compartment scores is needed
- Compartment scores already come from a common normalized pipeline and distribution
  alignment would remove genuine between-condition score differences
- The goal is differential interaction (loop) calling (e.g., Fit-Hi-C-based analysis)
  rather than compartment score comparison
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: dcHiC
  role: Implements step-1 partial SVD compartment calling followed by quantile normalization
    of compartment scores across all input Hi-C data as the prelude to step-2 differential
    testing
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:e3f53f4907372e1131a5dfcb0a7ed41217b1fe392414ec349cdc9b007366b070
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# quantile-normalization-of-compartment-scores

## Summary

dcHiC applies quantile normalization to compartment scores (PC1/PC2 from a partial SVD) across two or more Hi-C contact maps/replicates so that scores are directly comparable before multivariate differential testing. This normalization is the bridge between per-map PCA and downstream Mahalanobis-distance significance calling.

## When to use

Use when compartment scores (PC1/PC2 vectors) have been computed for multiple Hi-C datasets or replicates and must be made comparable across conditions prior to differential compartment analysis.

## When NOT to use

- Only a single Hi-C contact map is being analyzed and no cross-condition comparison of compartment scores is needed
- Compartment scores already come from a common normalized pipeline and distribution alignment would remove genuine between-condition score differences
- The goal is differential interaction (loop) calling (e.g., Fit-Hi-C-based analysis) rather than compartment score comparison

## Inputs

- Per-map compartment score matrices (PC1 or PC2 eigenvectors from dcHiC step-1 partial SVD) for each Hi-C dataset or replicate
- Binned genomic coordinates defining the score matrix rows (e.g., 100 kb bins as in the ESC/NPC analyses)

## Outputs

- Quantile-normalized compartment score matrix with comparable score distributions across all input Hi-C maps, ready for Mahalanobis-distance differential testing

## How to apply

After dcHiC's step-1 partial SVD (parallelized, memory-efficient Filebacked Big Matrix) computes principal components for each input Hi-C map, assemble the per-bin compartment score vectors from all maps into a binned score matrix. Apply quantile normalization across all maps/replicates so score distributions are aligned and comparable between conditions. The rationale is that raw PCA scores are not on a common scale, which would distort the multivariate distance computed in step 2. The normalized score matrix then feeds directly into the Mahalanobis-distance-based outlier test with chi-square p-values and IHW or Benjamini-Hochberg FDR correction, with dcHiC supporting resolutions up to 10 kb and pseudo-bulk profiles from as few as 100 cells per condition.

## Related tools

- **dcHiC** (Implements step-1 partial SVD compartment calling followed by quantile normalization of compartment scores across all input Hi-C data as the prelude to step-2 differential testing) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- After normalization, the compartment score distributions (quantiles) of each replicate/map are aligned across conditions, so no single map dominates the score range
- The normalized score matrix has one row per genomic bin per map, with bin coordinates matching the resolution used in step-1 PCA (up to 10 kb supported)
- Downstream Mahalanobis-distance p-values follow the expected chi-square distribution on the normalized scores; significant bins at FDR 10% (e.g., RefineY_Rconf90_FDR10 settings in the ESC/NPC outputs) are reproducible across reruns
- Normalization does not flip A/B compartment assignments — bins retain their compartment sign relative to eigenvector orientation

## Limitations

- Quantile normalization assumes comparable score distributions; strong global compartment shifts between conditions could be partially erased before differential testing
- For single-cell Hi-C, pseudo-bulk aggregation is required first, and genomic compartment identification remains limited by map sparsity even at coarse resolution
- Analysis demonstrated on human and mouse Hi-C data; applicability to other genomes/assay variants is asserted but not benchmarked in this work

## Evidence

- [intro] dcHiC uses quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates and a multivariate distance measure to estimate statistical significance of compartment differences: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Step-1 computes PCs then quantile-normalizes scores across all input data before step-2 distance calculation: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [intro] dcHiC implements a memory-efficient parallelized partial SVD to obtain the compartment eigenvectors that are subsequently normalized: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] dcHiC supports high-resolution compartment analysis on pseudo-bulk single-cell Hi-C data: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb and for pseudo-bulk Hi-C profiles from as few as 100 cells per condition"
- [abstract] dcHiC tool description confirming its scope for multi-dataset compartment comparison: "Here, we introduce dcHiC, which utilizes a multivariate distance measure to identify significant changes in compartmentalization among multiple contact maps"
