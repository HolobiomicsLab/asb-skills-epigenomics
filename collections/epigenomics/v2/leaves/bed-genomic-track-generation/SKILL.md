---
name: bed-genomic-track-generation
description: Use when Hi-C contact maps (bulk or pseudo-bulk single-cell, resolutions
  up to 10 kb) must be converted into per-bin compartment score tracks in a BED-like
  output, with automated PC1/PC2 and sign orientation per chromosome.
when_to_use_negative:
- Single-cell Hi-C maps used directly without pseudo-bulk aggregation — sparsity of
  single-cell contact maps makes compartment identification unreliable even at coarse
  resolution
- The goal is differential compartment testing between conditions — that is the downstream
  Mahalanobis-distance significance step, not track generation
- Resolutions finer than the supported range (dcHiC compartment analysis is stated
  for resolutions up to 10 kb; the README claims up to 5 kb for the optimized PCA)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: dcHiC
  role: Performs distance normalization, correlation transformation, partial SVD,
    automated PC/sign selection, and writes compartment_scores.bed.gz
  repo: https://github.com/ay-lab/dcHiC
- name: bigstatsr
  role: Provides the Filebacked Big Matrix (FBM) data structure enabling memory-efficient,
    parallelized partial SVD of chromosome correlation matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:bf0a78b3582c2e7127dcb09df8d6b9655c6713af8d171be3591ed4866f9995d5
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

# bed-genomic-track-generation

## Summary

Generates per-bin genomic tracks of A/B compartment scores as a BED-like table (compartment_scores.bed.gz) by computing principal components of distance-normalized Hi-C correlation matrices with an automated PC and sign selection. The sign is oriented so that GC-rich, gene-dense (A-type) bins carry positive scores.

## When to use

Use when Hi-C contact maps (bulk or pseudo-bulk single-cell, resolutions up to 10 kb) must be converted into per-bin compartment score tracks in a BED-like output, with automated PC1/PC2 and sign orientation per chromosome.

## When NOT to use

- Single-cell Hi-C maps used directly without pseudo-bulk aggregation — sparsity of single-cell contact maps makes compartment identification unreliable even at coarse resolution
- The goal is differential compartment testing between conditions — that is the downstream Mahalanobis-distance significance step, not track generation
- Resolutions finer than the supported range (dcHiC compartment analysis is stated for resolutions up to 10 kb; the README claims up to 5 kb for the optimized PCA)

## Inputs

- Chromosome-wise Hi-C contact maps at up to 10 kb resolution (bulk or pseudo-bulk single-cell)
- Reference tracks for sign orientation: gene density and GC content per genomic bin

## Outputs

- compartment_scores.bed.gz — BED-like table with one compartment score row per genomic bin per chromosome
- Quantile-normalized comparable compartment scores across input Hi-C maps/replicates

## How to apply

Load each chromosome's Hi-C contact map into a memory-efficient Filebacked Big Matrix (FBM via bigstatsr) at the chosen resolution (up to 10 kb, README claims support down to 5 kb). Distance-normalize by observed/expected at each genomic distance, transform into a correlation matrix, and compute only the first few singular vectors with a parallelized partial SVD. Select PC1 (or PC2 when PC1 does not capture the compartment pattern) and automatically reorient its sign so scores correlate positively with gene density and GC content, marking A-type compartments as positive. Write the per-chromosome score vectors as compartment_scores.bed.gz, one score row per genomic bin per chromosome. Subsequent quantile normalization (step 2 of dcHiC) makes scores comparable across multiple maps/replicates.

## Related tools

- **dcHiC** (Performs distance normalization, correlation transformation, partial SVD, automated PC/sign selection, and writes compartment_scores.bed.gz) — https://github.com/ay-lab/dcHiC
- **bigstatsr** (Provides the Filebacked Big Matrix (FBM) data structure enabling memory-efficient, parallelized partial SVD of chromosome correlation matrices)

## Examples

```
Rscript -e 'library(dcHiC); dchic("-f", "sample1.bedpe", "-r", 10000, "-o", "out", "--cores", 4)'
```

## Evaluation signals

- compartment_scores.bed.gz exists and parses as a BED-like table with exactly one score row per genomic bin per chromosome
- Sign orientation is correct: bins with high GC content and high gene density carry positive scores (A-type positive)
- PC1 is used by default; PC2 substitution is only applied when PC1 does not represent the compartment pattern, and the choice should be justified per sample
- Only the first few singular vectors are computed (partial SVD), not a full decomposition, confirming memory-efficient FBM-based execution
- Scores are quantile-normalized when multiple maps/replicates are input, yielding comparable distributions across samples

## Limitations

- Genomic compartment identification from single-cell Hi-C maps is limited by contact map sparsity, even at coarse resolution; pseudo-bulk aggregation (down to ~80-100 cells per condition) is required
- Analysis demonstrated only on human and mouse Hi-C data, though the method is claimed to be readily applicable to Hi-C variants
- Standard PCA-based compartment analysis examines each contact map individually and cannot by itself compare compartmentalization across multiple (>2) datasets

## Evidence

- [intro] Partial SVD with FBM: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Resolution and cell limits: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb and for pseudo-bulk Hi-C profiles from as few as 100 cells per condition"
- [intro] Normalization workflow: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [intro] Quantile normalization of scores: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [readme] Optimized PCA resolution claim: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
