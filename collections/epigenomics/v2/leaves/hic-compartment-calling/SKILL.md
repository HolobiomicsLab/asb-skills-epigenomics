---
name: hic-compartment-calling
description: Use when you need genome-wide A/B compartment scores from one or more
  Hi-C contact maps (bulk, pseudo-replicate, or pseudo-bulk single-cell) at resolutions
  from 100 kb down to 5-10 kb, or comparable scores across replicates/conditions.
when_to_use_negative:
- You need statistically significant differential compartmentalization between conditions
  rather than per-map compartment scores — use dcHiC's Mahalanobis-distance differential
  testing instead
- Single-cell Hi-C maps are too sparse to call compartments per cell — aggregate to
  pseudo-bulk first; the article notes single-cell compartment identification is limited
  by sparsity even at coarse resolution
- You only need significant chromatin interactions/loops rather than compartments
  — Fit-Hi-C, not compartment calling, is the appropriate tool
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: dcHiC
  role: 'Primary tool: performs distance normalization to a correlation matrix, parallelized
    partial SVD on a Filebacked Big Matrix, and quantile normalization of compartment
    scores'
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: 'Benchmark comparator: independent compartment scoring on the same ESC map
    for genome-wide Pearson correlation against dcHiC scores'
- name: CscoreTool
  role: 'Benchmark comparator: independent compartment calling on identical binned
    maps for concordance and runtime/memory benchmarking'
- name: IGV
  role: Visualization of compartment scores via dcHiC's standalone dynamic IGV web
    browser view
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:338a058a887149d5231695605c0e687f7f80246b041ae927086ee3b4cb251037
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - CscoreTool
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hic-compartment-calling

## Summary

Calls A/B compartments from Hi-C contact maps using distance normalization to a correlation matrix, partial SVD/PCA to extract PC1 compartment scores, and quantile normalization across maps — as implemented in dcHiC with a memory-efficient Filebacked Big Matrix and validated against HOMER and CscoreTool.

## When to use

Use when you need genome-wide A/B compartment scores from one or more Hi-C contact maps (bulk, pseudo-replicate, or pseudo-bulk single-cell) at resolutions from 100 kb down to 5-10 kb, or comparable scores across replicates/conditions.

## When NOT to use

- You need statistically significant differential compartmentalization between conditions rather than per-map compartment scores — use dcHiC's Mahalanobis-distance differential testing instead
- Single-cell Hi-C maps are too sparse to call compartments per cell — aggregate to pseudo-bulk first; the article notes single-cell compartment identification is limited by sparsity even at coarse resolution
- You only need significant chromatin interactions/loops rather than compartments — Fit-Hi-C, not compartment calling, is the appropriate tool

## Inputs

- Binned Hi-C contact map (e.g., mouse ESC Hi-C at 100 kb resolution, genome-wide)
- Hi-C maps with or without replicates; pseudo-bulk single-cell Hi-C profiles from as few as 100 cells per condition
- Reference compartment-score tracks for benchmarking (HOMER, CscoreTool) and ChIP-seq signal such as Lamin B1 Data track (optional)

## Outputs

- Per-bin compartment scores (PC1) per map/replicate after quantile normalization
- Pairwise method concordance Pearson correlations (e.g., dcHiC-HOMER r = 0.96, HOMER-CscoreTool r = 0.97, CscoreTool-dcHiC r = 0.98)
- Lamin B1 correlation R-values per method (dcHiC -0.91, HOMER -0.91, CscoreTool -0.89)

## How to apply

Start from a binned Hi-C contact map (e.g., mouse ESC at 100 kb resolution; dcHiC supports up to 10 kb and pseudo-bulk profiles from as few as 100 cells per condition). Perform distance normalization (observed/expected at each genomic distance), transform into a correlation matrix, and run a parallelized partial SVD on a Filebacked Big Matrix (FBM) to compute only the first few singular vectors needed for compartment analysis (PC1, or PC2 where appropriate). Follow with quantile normalization of compartment scores across all input Hi-C maps/replicates so scores are comparable. Validate concordance by computing genome-wide Pearson correlations against HOMER and CscoreTool scores on identical genomic bins, and optionally against orthogonal Lamin B1 Data signal (R-values around -0.9 expected). At 100 kb with full sequencing depth, expect dcHiC to use moderate memory (~0.34 Gb, less than HOMER's ~1.2 Gb) while running 22-33x faster than HOMER and 4-13x faster than CscoreTool.

## Related tools

- **dcHiC** (Primary tool: performs distance normalization to a correlation matrix, parallelized partial SVD on a Filebacked Big Matrix, and quantile normalization of compartment scores) — https://github.com/ay-lab/dcHiC
- **HOMER** (Benchmark comparator: independent compartment scoring on the same ESC map for genome-wide Pearson correlation against dcHiC scores)
- **CscoreTool** (Benchmark comparator: independent compartment calling on identical binned maps for concordance and runtime/memory benchmarking)
- **IGV** (Visualization of compartment scores via dcHiC's standalone dynamic IGV web browser view)

## Examples

```
conda install -c bioconda dchic  # then in R: run dchic compartment analysis on the binned ESC contact map at 100 kb (see demo/ directory of github.com/ay-lab/dcHiC)
```

## Evaluation signals

- Genome-wide Pearson correlation of dcHiC compartment scores vs HOMER on mouse ESC 100 kb bins reaches r = 0.96, and vs CscoreTool r = 0.98 (HOMER-CscoreTool r = 0.97)
- Correlation of each method's scores with Lamin B1 Data signal yields R-values near -0.9 (dcHiC -0.91, HOMER -0.91, CscoreTool -0.89)
- At 100 kb full depth, peak memory ordering is CscoreTool (~0.24 Gb) < dcHiC (~0.34 Gb) < HOMER (~1.2 Gb), with dcHiC speedups of 4-13x over CscoreTool and 22-33x over HOMER
- All pairwise comparisons use identical genomic bins — verify bin coordinate alignment between methods before correlating
- Compartment scores across replicates/conditions must be quantile-normalized to be comparable

## Limitations

- Genomic compartment identification from single-cell Hi-C maps is still limited by sparsity even at coarse resolution; pseudo-bulk aggregation is required
- Analysis demonstrated on human and mouse Hi-C data, though the authors note the method is readily applicable to Hi-C data or its variants
- Standard PCA-based compartment analysis is limited to analyzing each contact map individually — cross-map comparison requires the quantile normalization and multivariate steps beyond plain PCA

## Evidence

- [intro] dcHiC implements a parallelized partial singular value decomposition using a memory-efficient Filebacked Big Matrix (FBM) data structure, computing only the first few singular vectors needed for compartment analysis: [evidence span withheld — anchor status: not_found]
- [intro] dcHiC uses quantile normalization to obtain comparable compartment scores across maps: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Distance normalization to observed/expected then correlation matrix transformation precedes eigenvalue decomposition: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [intro] dcHiC supports compartment analysis up to 10 kb and pseudo-bulk from 100 cells per condition: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb and for pseudo-bulk Hi-C profiles from as few as 100 cells per condition"
- [readme] dcHiC runs pre-dominantly from R and Python with conda installation: [evidence span withheld — anchor status: external_doc]
- [abstract] dcHiC enables high-resolution compartment analysis and browser visualization: "dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering"
