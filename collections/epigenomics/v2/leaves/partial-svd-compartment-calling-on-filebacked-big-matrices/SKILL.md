---
name: partial-svd-compartment-calling-on-filebacked-big-matrices
description: Use when per-sample A/B compartment scores must be computed from Hi-C
  contact maps (resolutions down to 5-10 kb) and full eigen-decomposition is infeasible
  due to memory limits on large chromosome matrices.
when_to_use_negative:
- Input is single-cell Hi-C without pseudo-bulk aggregation — single-cell contact
  maps are too sparse for reliable compartment identification; aggregate ~100+ cells
  per condition first
- The goal is comparing compartmentalization across multiple (>2) Hi-C datasets rather
  than calling compartments per sample — use dcHiC's quantile normalization and Mahalanobis-distance
  differential step instead
- Resolution finer than ~5 kb is required — the FBM-based partial SVD is validated
  up to 5-10 kb
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: dcHiC
  role: Implements distance normalization, correlation-matrix transformation, and
    the partial SVD compartment call with automated PC/sign selection
  repo: https://github.com/ay-lab/dcHiC
- name: bigstatsr
  role: Provides the Filebacked Big Matrix (FBM) data structure over which the parallelized
    partial SVD runs memory-efficiently
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:20812bb19ddd7ae2cb95d7198e15d29f980aac2760a4ff86fcd16fa268e51311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - bigstatsr
  - dcHiC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# partial-svd-compartment-calling-on-filebacked-big-matrices

## Summary

Computes A/B compartment scores from Hi-C contact maps by distance-normalizing (observed/expected) each chromosome matrix, converting it to a correlation matrix, and running a parallelized partial SVD (only the first few singular vectors) over a memory-efficient Filebacked Big Matrix (FBM) via bigstatsr, with automated PC and sign selection based on GC content and gene density.

## When to use

Use when per-sample A/B compartment scores must be computed from Hi-C contact maps (resolutions down to 5-10 kb) and full eigen-decomposition is infeasible due to memory limits on large chromosome matrices.

## When NOT to use

- Input is single-cell Hi-C without pseudo-bulk aggregation — single-cell contact maps are too sparse for reliable compartment identification; aggregate ~100+ cells per condition first
- The goal is comparing compartmentalization across multiple (>2) Hi-C datasets rather than calling compartments per sample — use dcHiC's quantile normalization and Mahalanobis-distance differential step instead
- Resolution finer than ~5 kb is required — the FBM-based partial SVD is validated up to 5-10 kb

## Inputs

- Chromosome-wise Hi-C contact maps (e.g. .hic or cooler-style matrices) at 5-10 kb resolution
- Per-sample Hi-C contact matrix for each dataset/replicate being analyzed
- GC content and gene density tracks per genomic bin (used for PC/sign orientation)

## Outputs

- compartment_scores.bed.gz — BED-like table of compartment scores, one row per genomic bin per chromosome
- Per-chromosome compartment score vectors with A-type bins positive

## How to apply

Load each chromosome's Hi-C contact matrix at the chosen resolution (dcHiC supports analysis up to 5-10 kb). Distance-normalize by observed/expected at each genomic distance, then transform into a correlation matrix. Instead of a full eigen-decomposition, run dcHiC's parallelized partial SVD over a Filebacked Big Matrix (FBM from bigstatsr), computing only the first few singular vectors needed for compartment analysis. Select PC1 (or PC2 when PC1 does not capture compartment structure) and reorient its sign so that scores correlate positively with GC content and gene density, making A-type compartments positive. Write the per-chromosome score vectors as compartment_scores.bed.gz with one score row per genomic bin.

## Related tools

- **dcHiC** (Implements distance normalization, correlation-matrix transformation, and the partial SVD compartment call with automated PC/sign selection) — https://github.com/ay-lab/dcHiC
- **bigstatsr** (Provides the Filebacked Big Matrix (FBM) data structure over which the parallelized partial SVD runs memory-efficiently)

## Evaluation signals

- compartment_scores.bed.gz exists and parses as a BED-like table with one score row per genomic bin per chromosome
- Sign orientation is correct: bins with high GC content and gene density carry positive scores (A-type positive)
- Only the first few singular vectors are computed (partial SVD), not the full spectrum
- Each chromosome matrix was distance-normalized (observed/expected per genomic distance) and converted to a correlation matrix before SVD
- Scores are usable downstream for quantile normalization across maps/replicates without per-map scale artifacts

## Limitations

- Standard PCA-based compartment analysis handles each Hi-C contact map individually; cross-dataset comparison requires the downstream quantile normalization and Mahalanobis-distance steps
- Genomic compartment identification from raw single-cell Hi-C maps is unreliable due to sparsity; pseudo-bulk aggregation of at least ~80-100 cells per condition is needed
- Resolution is bounded — dcHiC's optimized PCA supports analysis up to 5 kb (paper states up to 10 kb), not finer

## Evidence

- [intro] Parallelized partial SVD via FBM: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Distance normalization workflow: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [readme] Resolution capability from README: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [intro] Resolution and pseudo-bulk support: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
