---
name: differential-compartment-significance-testing
description: Use when comparing compartmentalization across ≥2 bulk Hi-C maps (with
  replicates, e.g. ESC vs NPC at 100 kb) and you need per-bin significance calls for
  compartment changes, including within-A/within-B differences without compartment
  flips.
when_to_use_negative:
- Standard per-map PCA compartment calling when the goal is only A/B assignment for
  a single contact map, not cross-condition differential testing
- Differential loop/interaction detection between contact maps — use Fit-Hi-C differential
  interactions anchored in dcHiC-called compartments instead
- 'Single-cell Hi-C compartment calling without pseudo-bulk aggregation: sparsity
  of single-cell maps limits compartment identification (dcHiC needs pseudo-bulk profiles,
  from as few as ~80-100 cells per condition)'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: 'Runs the full pipeline: observed/expected normalization, correlation matrix,
    partial SVD PCA, quantile normalization of compartment scores, Mahalanobis-distance
    chi-square testing with FDR/IHW correction, and IGV/HTML output'
  repo: https://github.com/ay-lab/dcHiC
- name: IGV
  role: Standalone dynamic browser view for visualizing called differential compartments
- name: HOMER
  role: Comparison method for differential scores alongside dcHiC calls
- name: Fit-Hi-C
  role: Identifies differential loops anchored in significant differential compartments
  repo: https://github.com/ay-lab/fithic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:ca428e1e15f6ca492fe0714cab0a468364c3afeadf77b1b78e8d70a45d24666a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - Fit-Hi-C
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# differential-compartment-significance-testing

## Summary

dcHiC's statistical procedure for calling significant differential compartments between two or more Hi-C contact maps: PCA on the observed/expected-normalized correlation matrix, quantile normalization of compartment scores, and Mahalanobis-distance outlier testing with chi-square significance plus FDR/IHW correction.

## When to use

Use when comparing compartmentalization across ≥2 bulk Hi-C maps (with replicates, e.g. ESC vs NPC at 100 kb) and you need per-bin significance calls for compartment changes, including within-A/within-B differences without compartment flips.

## When NOT to use

- Standard per-map PCA compartment calling when the goal is only A/B assignment for a single contact map, not cross-condition differential testing
- Differential loop/interaction detection between contact maps — use Fit-Hi-C differential interactions anchored in dcHiC-called compartments instead
- Single-cell Hi-C compartment calling without pseudo-bulk aggregation: sparsity of single-cell maps limits compartment identification (dcHiC needs pseudo-bulk profiles, from as few as ~80-100 cells per condition)

## Inputs

- Hi-C contact maps at 100 kb resolution for two or more conditions with replicates (e.g. ESC 4 replicates, 230M-1.2B reads; NPC 3 replicates, 720M-1.5B reads)
- Down-sampled Hi-C contact maps (e.g. 10%-80% of original sequencing depth) when assessing depth robustness
- Replicate/filter configuration (e.g. RefineY_Rconf90_FDR10)
- Bin size / resolution setting (e.g. 100 kb)

## Outputs

- Per-bin Mahalanobis distances with chi-square p-values and FDR/IHW-adjusted significance calls
- Set of significant differential compartment bins (e.g. 1981 bins full-depth; 1906 recovered in 3v3 analysis)
- Standalone IGV/HTML visualization files (e.g. ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html)
- Depth-vs-recall tables such as downsampling_recall.csv

## How to apply

First compute distance normalization (observed/expected per genomic distance) for each Hi-C contact map, transform it into a correlation matrix, and run dcHiC's parallelized partial SVD (Filebacked Big Matrix) to obtain PC1/PC2 compartment scores at the chosen resolution (dcHiC supports up to 10 kb; the ESC-vs-NPC case study uses 100 kb). Next apply dcHiC's quantile normalization so compartment scores are comparable across all input maps/replicates, then for each genomic bin compute the Mahalanobis distance — a multivariate z-score measuring how outlying the bin is with respect to the compartment score distribution. Assign statistical significance via a chi-square test and apply multiple-testing correction (IHW or FDR; e.g. the RefineY_Rconf90_FDR10 setting for the ESC-vs-NPC 100 kb analysis). Extract significant differential compartment bins and, for robustness, intersect across replicate subsets (e.g. the 3v3 ESC-vs-NPC comparison excluding the 230M-read ESC replicate) before reporting output as HTML/IGV visualizations or tables like downsampling_recall.csv.

## Related tools

- **dcHiC** (Runs the full pipeline: observed/expected normalization, correlation matrix, partial SVD PCA, quantile normalization of compartment scores, Mahalanobis-distance chi-square testing with FDR/IHW correction, and IGV/HTML output) — https://github.com/ay-lab/dcHiC
- **IGV** (Standalone dynamic browser view for visualizing called differential compartments)
- **HOMER** (Comparison method for differential scores alongside dcHiC calls)
- **Fit-Hi-C** (Identifies differential loops anchored in significant differential compartments) — https://github.com/ay-lab/fithic

## Evaluation signals

- At 100 kb resolution, down-sampling ESC and NPC replicates to ≥40% of full sequencing depth retains over 80% recall of the full-depth differential compartment calls
- In the 3v3 ESC-vs-NPC analysis (excluding the 230M-read ESC replicate), 1906 of the 1981 significant bins are recovered — a sanity check on replicate-driven robustness
- Significance calls require chi-square p-values passing the FDR (e.g. FDR10) or IHW correction; uncorrected Mahalanobis distances alone are not sufficient
- Compartment scores must be quantile-normalized across all input maps/replicates before distance testing, otherwise scores are not comparable
- Results should reproduce with dcHiC's visualization outputs (e.g. ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html) matching the filter settings used

## Limitations

- Compartment identification from single-cell Hi-C maps is still limited by map sparsity; pseudo-bulk aggregation is required before differential analysis
- Over 10% of significant ESC-vs-NPC compartment differences fall in nucleolus-associated domains, but NAD analysis was reported only as initial/preliminary
- Compartment comparison across thousands of cells and tens of conditions is deferred to future extensions of dcHiC
- The demonstrated applications were limited to human and mouse Hi-C data, though the method is stated to be readily applicable to Hi-C data or its variants

## Evidence

- [intro] Mahalanobis outlier testing: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Quantile normalization: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Statistical correction: [evidence span withheld — anchor status: not_found]
- [methods] Replicable settings: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
- [readme] Scope of significant calls: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
