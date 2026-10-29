---
name: cross-method-compartment-score-comparison
description: Use when you have compartment-score tracks from dcHiC, HOMER, and/or
  CscoreTool over identical genomic bins of the same Hi-C map (e.g., mouse ESC at
  100 kb) and need to quantify cross-method concordance and lamin-association agreement.
when_to_use_negative:
- When the goal is detecting statistically significant differential compartments across
  two or more contact maps rather than benchmarking concordance of compartment scores
  from a single map — dcHiC's Mahalanobis-distance/chi-square significance testing
  is the appropriate mode there
- When single-cell Hi-C maps are too sparse for reliable compartment identification
  even at coarse resolution — single-cell sparsity is a known major limitation for
  compartment calling
- When contact maps are at resolutions finer than dcHiC's supported range or come
  from organisms outside the human/mouse data scope validated in this work
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Produces the query compartment-score track via partial SVD on a Filebacked
    Big Matrix with quantile normalization of compartment scores
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Reference compartment caller providing HOMER Differential Scores for the same
    ESC Hi-C map to correlate against dcHiC scores
- name: CscoreTool
  role: Reference compartment caller whose scores provide the second independent benchmark
    track for pairwise Pearson correlation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:9ff54567810e687ed5a7b4652c42cc2e62479d7130a55ddbfc64b1d14cbe5c5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - CscoreTool
  - HOMER
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-method-compartment-score-comparison

## Summary

Benchmarks compartment-score tracks from dcHiC against established compartment callers (HOMER and CscoreTool) on the same Hi-C contact map, quantifying genome-wide concordance by Pearson correlation and orthogonal validation against Lamin B1 data.

## When to use

Use when you have compartment-score tracks from dcHiC, HOMER, and/or CscoreTool over identical genomic bins of the same Hi-C map (e.g., mouse ESC at 100 kb) and need to quantify cross-method concordance and lamin-association agreement.

## When NOT to use

- When the goal is detecting statistically significant differential compartments across two or more contact maps rather than benchmarking concordance of compartment scores from a single map — dcHiC's Mahalanobis-distance/chi-square significance testing is the appropriate mode there
- When single-cell Hi-C maps are too sparse for reliable compartment identification even at coarse resolution — single-cell sparsity is a known major limitation for compartment calling
- When contact maps are at resolutions finer than dcHiC's supported range or come from organisms outside the human/mouse data scope validated in this work

## Inputs

- Mouse ESC Hi-C contact map at 100 kb resolution
- dcHiC compartment-score track (partial SVD on Filebacked Big Matrix, quantile-normalized scores)
- HOMER compartment scores (HOMER Differential Scores) on the same 100 kb bins
- CscoreTool compartment scores on the same 100 kb bins
- Genome-wide Lamin B1 Dam-ID signal track

## Outputs

- compartment_method_correlations.csv containing pairwise Pearson correlations (dcHiC-HOMER, HOMER-CscoreTool, CscoreTool-dcHiC) and per-method Lamin B1 R-values
- Genome-wide Pearson r values quantifying concordance between compartment-score tracks

## How to apply

Load the Hi-C contact map (e.g., mouse ESC at 100 kb resolution) and run dcHiC compartment calling, which performs a parallelized partial SVD on a memory-efficient Filebacked Big Matrix (FBM) followed by quantile normalization of compartment scores. Run HOMER compartment scoring (HOMER Differential Scores) and CscoreTool on the identical map so three independent compartment-score tracks cover the same genomic bins. Compute genome-wide Pearson correlations for each pairwise combination of the three methods' scores (dcHiC-HOMER, HOMER-CscoreTool, CscoreTool-dcHiC). Additionally correlate each method's compartment scores with the Lamin B1 Dam-ID signal genome-wide to obtain an R-value per method, then write all pairwise correlations and Lamin B1 R-values to compartment_method_correlations.csv.

## Related tools

- **dcHiC** (Produces the query compartment-score track via partial SVD on a Filebacked Big Matrix with quantile normalization of compartment scores) — https://github.com/ay-lab/dcHiC
- **HOMER** (Reference compartment caller providing HOMER Differential Scores for the same ESC Hi-C map to correlate against dcHiC scores)
- **CscoreTool** (Reference compartment caller whose scores provide the second independent benchmark track for pairwise Pearson correlation)

## Evaluation signals

- Genome-wide Pearson r between dcHiC and HOMER compartment scores should match the reported 0.96 within small numerical tolerance
- Genome-wide Pearson r between HOMER and CscoreTool should match the reported 0.97, and CscoreTool vs dcHiC the reported 0.98
- Lamin B1 R-values per method should match the reported benchmarks: dcHiC -0.91, HOMER -0.91, CscoreTool -0.89
- All three compartment-score tracks must be aligned to identical genomic bins at 100 kb resolution before correlations are computed
- compartment_method_correlations.csv should contain exactly three pairwise method correlations plus three Lamin B1 R-values

## Limitations

- Compartment identification from single-cell Hi-C maps remains a major issue due to sparsity of single-cell contact maps, even at coarse resolution
- Analysis in this work was limited to human and mouse Hi-C data, although the method is readily applicable to other Hi-C data or its variants
- Standard PCA-based compartment analysis is limited to analyzing each Hi-C contact map individually, so cross-map differential analysis needs dcHiC's multivariate framework rather than this concordance benchmark

## Evidence

- [intro] dcHiC implements partial SVD with quantile normalization of compartment scores: "Our method implements a parallelized partial singular value decomposition (SVD) that uses a memory-efficient data structure (Filebacked Big Matrix or FBM)"
- [intro] Quantile normalization makes compartment scores comparable across maps: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [methods] HOMER differential scores were computed as a comparison baseline: "HOMER Differential Scores (for comparison)"
- [readme] dcHiC is the compartment analysis tool under benchmark: "dcHiC is a tool for differential compartment analysis of Hi-C datasets."
- [abstract] dcHiC's core statistical machinery for compartmentalization analysis: "Here, we introduce dcHiC, which utilizes a multivariate distance measure to identify significant changes in compartmentalization among multiple contact maps"
