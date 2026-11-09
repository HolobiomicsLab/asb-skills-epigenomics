---
name: eigenvector-principal-component-analysis
description: Use when you need compartment scores (A/B calls) from one or more Hi-C
  contact maps, especially at high resolution (up to 5-10 kb) or across multiple maps/replicates
  needing comparable scores.
when_to_use_negative:
- The goal is detecting significant compartment differences between maps rather than
  calling compartments per map — use dcHiC's Mahalanobis-distance differential step
  instead of plain PCA
- Input is single-cell Hi-C individual cells, where sparsity makes compartment identification
  unreliable — dcHiC recommends pseudo-bulk aggregation first (down to ~80-100 cells
  per condition)
- You only need significant chromatin interactions rather than compartmentalization
  — Fit-Hi-C, not eigenvector PCA, is the appropriate tool
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: dcHiC
  role: Runs the optimized partial SVD-based PCA compartment calling on Filebacked
    Big Matrix structures with quantile normalization, up to 5 kb resolution
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Independent compartment scoring method used as a benchmark to correlate PC-derived
    scores against (HOMER Differential Scores)
- name: CscoreTool
  role: Third compartment-calling method used for cross-method concordance checks
    of the eigenvector-based scores
- name: IGV
  role: Used by dcHiC to render standalone dynamic browser views of compartment scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:376db93581bf47a4e1d1c3bc69e46cf832c9a2cc0624b50a4f23afedddd80856
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - CscoreTool
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# eigenvector-principal-component-analysis

## Summary

Computes A/B compartments from a Hi-C contact map by distance normalization, conversion to a correlation matrix, and eigenvector decomposition, retaining PC1/PC2 as compartment scores. dcHiC implements this as a parallelized partial SVD on a memory-efficient Filebacked Big Matrix followed by quantile normalization, enabling analysis up to 5 kb resolution.

## When to use

Use when you need compartment scores (A/B calls) from one or more Hi-C contact maps, especially at high resolution (up to 5-10 kb) or across multiple maps/replicates needing comparable scores.

## When NOT to use

- The goal is detecting significant compartment differences between maps rather than calling compartments per map — use dcHiC's Mahalanobis-distance differential step instead of plain PCA
- Input is single-cell Hi-C individual cells, where sparsity makes compartment identification unreliable — dcHiC recommends pseudo-bulk aggregation first (down to ~80-100 cells per condition)
- You only need significant chromatin interactions rather than compartmentalization — Fit-Hi-C, not eigenvector PCA, is the appropriate tool

## Inputs

- Hi-C contact map (e.g., .hic or cooler format) at a chosen bin resolution such as 100 kb or 5 kb
- Multiple Hi-C maps or replicates when comparable compartment scores across datasets are needed
- Optional genome-wide epigenomic track (e.g., Lamin B1 Dam-ID signal) for validation of compartment orientation

## Outputs

- Per-bin compartment score track (PC1 or PC2 values per genomic bin, quantile-normalized across maps)
- Pairwise genome-wide Pearson correlation matrix of compartment scores across methods (e.g., dcHiC-HOMER, HOMER-CscoreTool, CscoreTool-dcHiC)
- Correlation (R-value) of each method's compartment scores with Lamin B1 signal, e.g., written to compartment_method_correlations.csv

## How to apply

Start from the Hi-C contact map (e.g., mouse ESC Hi-C at 100 kb) and perform distance normalization (observed/expected for each genomic distance) per chromosome, then transform into a correlation matrix and run eigenvector decomposition, selecting PC1 (or PC2) as the compartment score track. In dcHiC, this is executed as a parallelized partial singular value decomposition on a Filebacked Big Matrix (FBM), computing only the first few singular vectors needed, which makes analysis feasible up to 5 kb resolution. Follow the eigenvector step with quantile normalization so compartment scores are comparable across two or more Hi-C maps or replicates. For benchmarking, compute genome-wide Pearson correlations between methods' scores — the paper reports r = 0.96 (dcHiC-HOMER), 0.97 (HOMER-CscoreTool), and 0.98 (CscoreTool-dcHiC) on mouse ESC at 100 kb — and correlate each track with Lamin B1 Dam-ID signal (R-values about -0.91 to -0.89).

## Related tools

- **dcHiC** (Runs the optimized partial SVD-based PCA compartment calling on Filebacked Big Matrix structures with quantile normalization, up to 5 kb resolution) — https://github.com/ay-lab/dcHiC
- **HOMER** (Independent compartment scoring method used as a benchmark to correlate PC-derived scores against (HOMER Differential Scores))
- **CscoreTool** (Third compartment-calling method used for cross-method concordance checks of the eigenvector-based scores)
- **IGV** (Used by dcHiC to render standalone dynamic browser views of compartment scores)

## Examples

```
Rscript -e 'library(dchic); dchic::runPCA(contact_map="ESC_100kb.hic", resolution=100000, quantnorm=TRUE)'
```

## Evaluation signals

- Genome-wide Pearson correlation of dcHiC vs HOMER scores should be r = 0.96 at 100 kb on mouse ESC Hi-C
- HOMER-CscoreTool correlation should be r = 0.97 and CscoreTool-dcHiC r = 0.98 within small numerical tolerance
- Each method's scores correlated with Lamin B1 Data should give R-values near dcHiC -0.91, HOMER -0.91, CscoreTool -0.89
- Sign of PC1/PC2 should be checked so A compartments align with Lamin B1-depleted (active) regions; a global sign flip means scores need inversion
- Quantile normalization should yield comparable score distributions across all input maps before any cross-map comparison

## Limitations

- Standard PCA-based compartment analysis is limited to analyzing each Hi-C contact map individually; cross-map comparison requires quantile normalization as in dcHiC
- Genomic compartment identification from single-cell Hi-C maps is still a major issue due to sparsity, even at coarse resolution
- Analysis demonstrated on human and mouse data; applicability to very large numbers of conditions/cells was deferred to future extensions of dcHiC

## Evidence

- [intro] dcHiC implements a parallelized partial singular value decomposition using a memory-efficient Filebacked Big Matrix (FBM) data structure: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome: "this process involves distance normalization (observed/expected for each genomic distance) of the Hi-C contact map for each chromosome"
- [readme] Optimized PCA calculations (faster + capable of analysis up to 5kb resolution): "- Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [intro] compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb and for pseudo-bulk Hi-C profiles from as few as 100 cells per condition"
