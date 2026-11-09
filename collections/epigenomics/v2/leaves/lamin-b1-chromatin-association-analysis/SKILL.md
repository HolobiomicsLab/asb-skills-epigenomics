---
name: lamin-b1-chromatin-association-analysis
description: Use when you need to validate A/B compartment score tracks (e.g., dcHiC
  PC1 scores at 100 kb) against an orthogonal Lamin B1 DamID/Data signal track via
  genome-wide Pearson correlation, or to benchmark concordance between compartment-calling
  methods.
when_to_use_negative:
- Input contact maps are single-cell Hi-C at fine resolution where sparsity makes
  compartments unreliable — the paper notes compartment identification from single-cell
  maps 'is still a major issue in this field due to sparsity'
- The analysis goal is differential compartment significance testing between conditions
  rather than orthogonal validation of compartment calls — use dcHiC's Mahalanobis-distance/chi-square
  differential testing instead
- No orthogonal chromatin signal (e.g., LaminB1 DamID) is available for the same genome
  assembly and binning, so the correlation cannot be computed
tools:
- name: dcHiC
  role: Produces quantile-normalized compartment scores via partial SVD on a Filebacked
    Big Matrix; one of the three score tracks correlated with LaminB1 signal
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Independent compartment scoring (HOMER Differential Scores used for comparison)
    on the same ESC map, yielding a second score track for LaminB1 correlation
- name: CscoreTool
  role: Third independent compartment-calling method whose scores are correlated with
    LaminB1 signal and with the other two methods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:922f0fcf605cf3980aca3018c6e96bbc420fa51481fef9643cea10f7f2676939
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - CscoreTool
  - HOMER
  - dcHiC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lamin-b1-chromatin-association-analysis

## Summary

This skill orthogonal-validates A/B compartment calls by correlating genome-wide compartment scores (from dcHiC, HOMER, and CscoreTool on the mouse ESC Hi-C map at 100 kb resolution) against Lamin B1 association signal, yielding per-method R-values. Lamina association of B-type compartments is expected to produce strongly negative correlations, so the R-value doubles as a biological sanity check of compartment-score polarity.

## When to use

Use when you need to validate A/B compartment score tracks (e.g., dcHiC PC1 scores at 100 kb) against an orthogonal Lamin B1 DamID/Data signal track via genome-wide Pearson correlation, or to benchmark concordance between compartment-calling methods.

## When NOT to use

- Input contact maps are single-cell Hi-C at fine resolution where sparsity makes compartments unreliable — the paper notes compartment identification from single-cell maps 'is still a major issue in this field due to sparsity'
- The analysis goal is differential compartment significance testing between conditions rather than orthogonal validation of compartment calls — use dcHiC's Mahalanobis-distance/chi-square differential testing instead
- No orthogonal chromatin signal (e.g., LaminB1 DamID) is available for the same genome assembly and binning, so the correlation cannot be computed

## Inputs

- mouse ESC Hi-C contact map at 100 kb resolution
- per-bin compartment score tracks from dcHiC (quantile-normalized PC1), HOMER, and CscoreTool on identical 100 kb genomic bins
- genome-wide LaminB1 Data signal track (binned to matching 100 kb bins)

## Outputs

- compartment_method_correlations.csv containing pairwise method-method Pearson correlations (dcHiC-HOMER, HOMER-CscoreTool, CscoreTool-dcHiC) and per-method LaminB1 R-values

## How to apply

First obtain compartment score tracks for identical genomic bins from each method: run dcHiC's partial SVD on a Filebacked Big Matrix (FBM) with quantile normalization of compartment scores, plus HOMER differential/compartment scoring and CscoreTool on the same mouse ESC 100 kb map. Load the LaminB1 Data signal track and bin it (or rebin it) to exactly match the 100 kb compartment bins. Compute genome-wide Pearson correlations between each method's scores and the LaminB1 signal, tracking the sign carefully — sign flips from PC1 orientation must be corrected before comparing R-values. Report the per-method LaminB1 R-values alongside the pairwise method-method correlations (dcHiC-HOMER, HOMER-CscoreTool, CscoreTool-dcHiC) in compartment_method_correlations.csv. Rationale: the paper positions lamin association as orthogonal validation of compartmentalization biology, so strong negative R-values confirm that B compartments coincide with lamina-proximal chromatin.

## Related tools

- **dcHiC** (Produces quantile-normalized compartment scores via partial SVD on a Filebacked Big Matrix; one of the three score tracks correlated with LaminB1 signal) — https://github.com/ay-lab/dcHiC
- **HOMER** (Independent compartment scoring (HOMER Differential Scores used for comparison) on the same ESC map, yielding a second score track for LaminB1 correlation)
- **CscoreTool** (Third independent compartment-calling method whose scores are correlated with LaminB1 signal and with the other two methods)

## Examples

```
r <- cor(dchic_scores$lamin_scores, laminb1_signal, method="pearson")  # per-method R vs LaminB1, alongside cor(dchic, homer) / cor(homer, cscore) / cor(cscore, dchic)
```

## Evaluation signals

- Pairwise method-method Pearson correlations on the mouse ESC 100 kb map should match r = 0.96 (dcHiC-HOMER), r = 0.97 (HOMER-CscoreTool), and r = 0.98 (CscoreTool-dcHiC) within a small numerical tolerance
- Per-method LaminB1 R-values should reproduce dcHiC -0.91, HOMER -0.91, and CscoreTool -0.89 within tolerance
- All LaminB1 R-values must be negative after any PC1 sign correction, since B compartments should associate with the lamina
- All three compartment score tracks and the LaminB1 track must be binned to identical 100 kb genomic bins with matched genome assembly before correlation
- compartment_method_correlations.csv should contain exactly three pairwise method correlations and three LaminB1 R-values (one per method)

## Limitations

- Correlation sign depends on PC1 orientation, which is arbitrary; negative R-values are only meaningful after consistent sign alignment across methods
- The published benchmark was restricted to human and mouse Hi-C data ('Although we focused on human and mouse Hi-C data in this work, our method is readily applicable...'), so R-value transfer to other species is untested
- LaminB1 R-values (-0.89 to -0.91) are close but not identical across methods, so exact reproduction depends on matching bin resolution (100 kb) and normalization choices (e.g., quantile normalization of dcHiC scores)

## Evidence

- [abstract] lamin association as orthogonal validation: "dcHiC reported regions with dynamically regulated genes associated with cell identity, along with correlated changes in chromatin states, subcompartments, replication timing and lamin association"
- [intro] dcHiC compartment score computation: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [intro] SVD implementation detail: [evidence span withheld — anchor status: not_found]
- [readme] resolution capability of dcHiC PCA: "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
- [abstract] orthogonal validation framing: "we show its effectiveness and sensitivity in detecting biologically relevant changes, including those orthogonally"
