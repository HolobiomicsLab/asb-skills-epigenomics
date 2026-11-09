---
name: hic-distance-normalization-to-correlation-matrices
description: Use when raw chromosome-wise Hi-C contact maps (up to 10 kb resolution)
  must be prepared for PCA/SVD compartment calling, so that distance-decay effects
  are removed before eigen-decomposition.
when_to_use_negative:
- The goal is differential interaction / loop significance testing (that is Fit-Hi-C's
  role), not compartment computation
- The data are sparse single-cell Hi-C maps without prior pseudo-bulk aggregation,
  where compartment identification is unreliable due to sparsity
- Compartment scores have already been computed and quantile-normalized across samples
  (proceed directly to Mahalanobis-distance differential analysis)
tools:
- name: dcHiC
  role: Implements the observed/expected distance normalization, correlation-matrix
    transformation, and downstream partial SVD per chromosome
  repo: https://github.com/ay-lab/dcHiC
- name: bigstatsr
  role: Provides the Filebacked Big Matrix (FBM) data structure backing the memory-efficient
    parallelized partial SVD over the correlation matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:e039036eb3adfc372fa8d018fe3f1e669b8671db5a6109f5d8b57f2eb3b72d87
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
yes_rate: 1.0
n_claims: 10
---

# Hi-C distance normalization to correlation matrices

## Summary

Preprocessing step in dcHiC's compartment pipeline: each chromosome-wise Hi-C contact matrix is distance-normalized by observed/expected at each genomic distance and converted into a correlation matrix, which is the standard input for eigen/SVD-based A/B compartment scoring.

## When to use

Use when raw chromosome-wise Hi-C contact maps (up to 10 kb resolution) must be prepared for PCA/SVD compartment calling, so that distance-decay effects are removed before eigen-decomposition.

## When NOT to use

- The goal is differential interaction / loop significance testing (that is Fit-Hi-C's role), not compartment computation
- The data are sparse single-cell Hi-C maps without prior pseudo-bulk aggregation, where compartment identification is unreliable due to sparsity
- Compartment scores have already been computed and quantile-normalized across samples (proceed directly to Mahalanobis-distance differential analysis)

## Inputs

- Chromosome-wise Hi-C contact maps (contact count matrices) at up to 10 kb resolution
- Chosen bin resolution (e.g., 5 kb or 10 kb)

## Outputs

- Distance-normalized, correlation-transformed chromosome matrices suitable for partial SVD / compartment scoring

## How to apply

For each chromosome, load the Hi-C contact map into a matrix (dcHiC supports a memory-efficient Filebacked Big Matrix) at the chosen resolution (up to 10 kb). Apply observed/expected distance normalization for each genomic distance so contact enrichment is judged relative to the expected distance-decay curve. Transform the normalized matrix into a correlation matrix, since compartments are derived from covariation of rows/columns rather than raw contact counts. This correlation matrix is then fed to a parallelized partial SVD (via bigstatsr/FBM) that computes only the first few singular vectors, from which PC1 (or PC2) is selected and sign-oriented against GC content and gene density.

## Related tools

- **dcHiC** (Implements the observed/expected distance normalization, correlation-matrix transformation, and downstream partial SVD per chromosome) — https://github.com/ay-lab/dcHiC
- **bigstatsr** (Provides the Filebacked Big Matrix (FBM) data structure backing the memory-efficient parallelized partial SVD over the correlation matrices)

## Evaluation signals

- Each chromosome matrix has been normalized per genomic distance (observed/expected) before transformation, not used raw
- The transformed matrix is a correlation matrix (diagonal ≈ 1, entries are Pearson-type correlations between bin vectors)
- Only the first few singular vectors are computed (partial SVD), not a full decomposition
- Resulting compartment scores can be sign-oriented so bins with high GC content and gene density carry positive (A-type) scores
- Output is parseable as one score per genomic bin per chromosome (BED-like table, e.g., compartment_scores.bed.gz)

## Limitations

- Analysis is supported at resolutions up to 10 kb (README advertises optimized PCA up to 5 kb resolution)
- Genomic compartment identification from individual single-cell Hi-C maps is limited by contact-map sparsity; pseudo-bulk aggregation (as few as ~80–100 cells per condition) is needed first
- PCA-based compartment analysis handles each Hi-C contact map individually; cross-sample comparison requires dcHiC's subsequent quantile normalization and differential testing

## Evidence

- [intro] Distance normalization is applied per genomic distance before correlation transformation: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [intro] The correlation matrix feeds a parallelized partial SVD over an FBM structure: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [readme] dcHiC's PCA is optimized to run at up to 5 kb resolution: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [intro] Compartment analysis is supported at resolutions up to 10 kb: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
