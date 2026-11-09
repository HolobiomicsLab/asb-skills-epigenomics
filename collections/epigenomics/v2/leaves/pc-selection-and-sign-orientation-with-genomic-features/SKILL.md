---
name: pc-selection-and-sign-orientation-with-genomic-features
description: Use when computing A/B compartment scores from a Hi-C contact map and
  you need to decide which eigenvector (PC1 vs PC2) is the compartment component and
  which sign corresponds to the A-type compartment.
when_to_use_negative:
- The goal is to compare compartmentalization across multiple (>2) Hi-C datasets rather
  than to compute per-sample compartment scores — use dcHiC's quantile normalization
  plus Mahalanobis-distance differential step instead
- Input is single-cell Hi-C contact maps at single-cell (not pseudo-bulk) resolution
  — compartment identification is limited by sparsity of single-cell contact maps
  even at coarse resolution
- The contact map resolution is coarser than needed or finer than supported — dcHiC
  compartment analysis is validated at resolutions up to 10 kb (README mentions up
  to 5 kb for optimized PCA)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: dcHiC
  role: Performs distance normalization, correlation-matrix transformation, partial
    SVD, and automated PC/sign selection with gene density and GC content for compartment
    scoring
  repo: https://github.com/ay-lab/dcHiC
- name: bigstatsr
  role: Provides the Filebacked Big Matrix (FBM) data structure backing the memory-efficient
    parallelized partial SVD
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:02312428dd60ed3ce7484a6e82ad9dc7411e260e600a35c7a0a19e7868919ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - bigstatsr
  - dcHiC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pc-selection-and-sign-orientation-with-genomic-features

## Summary

Selects the principal component (PC1 or PC2) representing A/B compartments from a Hi-C contact map and orients its sign so compartment scores correlate positively with GC content and gene density, as implemented in dcHiC via a parallelized partial SVD on a Filebacked Big Matrix.

## When to use

Use when computing A/B compartment scores from a Hi-C contact map and you need to decide which eigenvector (PC1 vs PC2) is the compartment component and which sign corresponds to the A-type compartment.

## When NOT to use

- The goal is to compare compartmentalization across multiple (>2) Hi-C datasets rather than to compute per-sample compartment scores — use dcHiC's quantile normalization plus Mahalanobis-distance differential step instead
- Input is single-cell Hi-C contact maps at single-cell (not pseudo-bulk) resolution — compartment identification is limited by sparsity of single-cell contact maps even at coarse resolution
- The contact map resolution is coarser than needed or finer than supported — dcHiC compartment analysis is validated at resolutions up to 10 kb (README mentions up to 5 kb for optimized PCA)

## Inputs

- Hi-C contact map per sample at the chosen bin resolution (up to 10 kb, .hic or similar format supported by dcHiC)
- Chromosomal genomic-bin partition of the reference genome
- Gene density and GC content tracks per genomic bin (used for sign orientation)

## Outputs

- compartment_scores.bed.gz — BED-like table with one compartment score row per genomic bin per chromosome, sign-oriented so A-type (GC-rich, gene-dense) bins are positive

## How to apply

Load the input Hi-C contact maps at the chosen resolution (up to 10 kb supported) into chromosome-wise matrices within dcHiC. Distance-normalize each chromosome matrix by observed/expected at each genomic distance, then transform it into a correlation matrix. Compute only the first few singular vectors using dcHiC's parallelized partial SVD implemented over a memory-efficient Filebacked Big Matrix (FBM, bigstatsr). Select PC1 (or PC2 when appropriate, e.g. when PC1 tracks a technical or chromosomal artifact) as the compartment component, and orient its sign so that scores correlate positively with GC content and gene density, making A-type compartments positive. Write the resulting per-chromosome compartment score vectors as a BED-like table (compartment_scores.bed.gz) before downstream quantile normalization and differential analysis.

## Related tools

- **dcHiC** (Performs distance normalization, correlation-matrix transformation, partial SVD, and automated PC/sign selection with gene density and GC content for compartment scoring) — https://github.com/ay-lab/dcHiC
- **bigstatsr** (Provides the Filebacked Big Matrix (FBM) data structure backing the memory-efficient parallelized partial SVD)

## Examples

```
Rscript dchic_compartment.R --resolution 10000 --output compartment_scores.bed.gz
```

## Evaluation signals

- compartment_scores.bed.gz exists and parses as a BED-like table with one score row per genomic bin per chromosome
- Sign orientation is correct: bins with high GC content and gene density carry positive compartment scores (A-type positive)
- Only the first few singular vectors were computed (partial SVD), not a full decomposition of the correlation matrix
- Each chromosome matrix was distance-normalized (observed/expected per genomic distance) before the correlation-matrix transformation
- Scores are comparable across samples once quantile normalization is subsequently applied across all input Hi-C maps

## Limitations

- Genomic compartment identification from single-cell Hi-C maps is limited by sparsity of single-cell contact maps, even at coarse resolution
- PC1 is not always the compartment signal — selection must fall back to PC2 when appropriate, requiring the automated correlation check with gene density and GC content
- Analysis in this work was limited to human and mouse Hi-C data, though the method is described as readily applicable to Hi-C variants

## Evidence

- [intro] Automated PC/sign selection rationale: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Workflow: normalization and PC1/PC2 selection: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [intro] Sign orientation with genomic features: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Resolution support: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
- [readme] Optimized PCA capability from README: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
