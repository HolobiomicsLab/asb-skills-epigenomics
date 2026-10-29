---
name: chromosome-wise-hic-matrix-processing
description: Use when per-chromosome Hi-C contact maps (bulk or pseudo-bulk, resolutions
  up to 10 kb, or 5 kb with optimized PCA) need to be converted into A/B compartment
  score tracks for downstream differential analysis.
when_to_use_negative:
- The goal is differential compartment testing across samples — this skill only produces
  per-sample scores; the Mahalanobis-distance/chi-square differential step operates
  on its output afterwards.
- Input is single-cell Hi-C without pseudo-bulk aggregation — the paper notes genomic
  compartment identification from individual single-cell maps is still limited by
  sparsity, so first aggregate to pseudo-bulk (as few as ~100 cells per condition).
tools:
- name: dcHiC
  role: Executes the chromosome-wise distance normalization, correlation-matrix transformation,
    partial SVD, and automated PC/sign selection to produce compartment_scores.bed.gz
  repo: https://github.com/ay-lab/dcHiC
- name: bigstatsr
  role: Provides the Filebacked Big Matrix (FBM) data structure over which the memory-efficient
    parallelized partial SVD is computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:c20771c324df8f7ac386fa6c66ddde1500d85012a370d915d7eadc1f51a6363b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - bigstatsr
  - dcHiC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# chromosome-wise-hic-matrix-processing

## Summary

Performs chromosome-wise Hi-C contact-map preprocessing — observed/expected distance normalization, transformation into a correlation matrix, and a memory-efficient parallelized partial SVD via a Filebacked Big Matrix — to yield A/B compartment scores, with automated selection of PC1 (or PC2) and sign orientation against GC content and gene density.

## When to use

Use when per-chromosome Hi-C contact maps (bulk or pseudo-bulk, resolutions up to 10 kb, or 5 kb with optimized PCA) need to be converted into A/B compartment score tracks for downstream differential analysis.

## When NOT to use

- The goal is differential compartment testing across samples — this skill only produces per-sample scores; the Mahalanobis-distance/chi-square differential step operates on its output afterwards.
- Input is single-cell Hi-C without pseudo-bulk aggregation — the paper notes genomic compartment identification from individual single-cell maps is still limited by sparsity, so first aggregate to pseudo-bulk (as few as ~100 cells per condition).

## Inputs

- Hi-C contact map per sample at chosen bin resolution (e.g., .hic, .cool, or matrix files loadable into dcHiC), chromosome-wise
- Chosen bin resolution parameter (up to 10 kb supported; 5 kb with optimized PCA)

## Outputs

- compartment_scores.bed.gz — BED-like table with one compartment score row per genomic bin per chromosome
- Per-chromosome compartment score vectors used for downstream quantile normalization

## How to apply

Load the input Hi-C contact maps at the chosen resolution (up to 10 kb supported; dcHiC's optimized PCA supports up to 5 kb) into chromosome-wise matrices. Distance-normalize each chromosome matrix by observed/expected at each genomic distance, then transform it into a correlation matrix. Compute only the first few singular vectors using a parallelized partial SVD implemented over a memory-efficient Filebacked Big Matrix (FBM, bigstatsr), rather than a full eigendecomposition. Select PC1 (or PC2 when appropriate) as the compartment component and orient its sign so scores correlate positively with GC content and gene density, so that A-type compartments carry positive scores. Write the per-chromosome score vectors as compartment_scores.bed.gz for downstream quantile normalization and differential testing.

## Related tools

- **dcHiC** (Executes the chromosome-wise distance normalization, correlation-matrix transformation, partial SVD, and automated PC/sign selection to produce compartment_scores.bed.gz) — https://github.com/ay-lab/dcHiC
- **bigstatsr** (Provides the Filebacked Big Matrix (FBM) data structure over which the memory-efficient parallelized partial SVD is computed)

## Evaluation signals

- compartment_scores.bed.gz exists and parses as a BED-like table with exactly one score row per genomic bin per chromosome.
- Sign orientation is correct: bins with high GC content and gene density carry positive scores (A-type compartments positive).
- Only the first few singular vectors were computed (partial SVD over FBM), not a full eigendecomposition of the correlation matrix.
- Choice of PC1 vs PC2 as the compartment component is documented per chromosome/sample, since dcHiC employs automated selection.
- Scores are computed per chromosome after observed/expected distance normalization, not on a whole-genome matrix.

## Limitations

- Compartment analysis resolution is limited: dcHiC supports up to 10 kb (README reports optimized PCA capable of analysis up to 5 kb resolution).
- Compartment identification from individual single-cell Hi-C maps remains unreliable due to contact-map sparsity; pseudo-bulk aggregation (demonstrated at ~80–100 cells per condition) is required first.

## Evidence

- [intro] Distance normalization, correlation transform, partial SVD: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [intro] Partial SVD over FBM: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Resolution support: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
- [readme] README: optimized PCA resolution: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [readme] README: dcHiC runs from R and Python: [evidence span withheld — anchor status: external_doc]
