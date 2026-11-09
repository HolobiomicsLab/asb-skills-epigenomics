---
name: compartment-flip-annotation-genomic-bins
description: Use when a pairwise Hi-C comparison (e.g., ESC vs NPC at 100 kb bins)
  yields dcHiC significance calls and each significant bin must be labeled as A->B
  flip, B->A flip, or within-compartment change, with flip fractions and HOMER concordance
  tallied.
when_to_use_negative:
- Analysis of a single contact map with no comparative condition — no differential
  call or flip can be defined; use standard PCA-based compartment calling instead
- Pseudo-bulk single-cell Hi-C with far fewer than ~80–100 cells per condition, where
  compartment identification is unreliable due to map sparsity
- When the goal is significance of individual chromatin interactions rather than compartmentalization
  changes — that is Fit-Hi-C's differential interaction domain, not bin-level flip
  annotation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: dcHiC
  role: Computes PC1 compartment scores via partial SVD, quantile-normalizes them
    across replicates, and produces Mahalanobis/chi-square/IHW significance calls
    whose bins are then annotated as A->B, B->A, or within-compartment changes
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Provides orthogonal differential compartment scores whose 3,042 ESC/NPC differential
    bins are intersected with dcHiC's calls (±1 bin slack) to quantify method concordance
- name: IGV
  role: Underlies dcHiC's standalone HTML browser output used to visualize and sanity-check
    the per-bin differential compartment and flip annotations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:6dcebaceccfa90bb5454983facf8595497e99747013f75ff8097faf8b89b8306
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - HOMER
  - IGV
  - dcHiC
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# compartment-flip-annotation-genomic-bins

## Summary

Annotates dcHiC's significant differential compartment bins (100 kb, FDR < 0.1) in a pairwise mouse ESC vs NPC Hi-C comparison as A->B flips, B->A flips, or within-compartment changes, then cross-checks them against HOMER's 3,042 differential bins (1,355 overlapping with ±1 bin slack).

## When to use

Use when a pairwise Hi-C comparison (e.g., ESC vs NPC at 100 kb bins) yields dcHiC significance calls and each significant bin must be labeled as A->B flip, B->A flip, or within-compartment change, with flip fractions and HOMER concordance tallied.

## When NOT to use

- Analysis of a single contact map with no comparative condition — no differential call or flip can be defined; use standard PCA-based compartment calling instead
- Pseudo-bulk single-cell Hi-C with far fewer than ~80–100 cells per condition, where compartment identification is unreliable due to map sparsity
- When the goal is significance of individual chromatin interactions rather than compartmentalization changes — that is Fit-Hi-C's differential interaction domain, not bin-level flip annotation

## Inputs

- ESC and NPC Hi-C contact maps binned at 100 kb, including replicate maps for each condition
- dcHiC quantile-normalized PC1 compartment scores per 100 kb bin across all ESC/NPC maps
- dcHiC differential significance calls (Mahalanobis distance, chi-square p-values, IHW-corrected FDR < 0.1 under RefineY_Rconf90_FDR10 settings)
- HOMER differential compartment scores for the ESC vs NPC comparison

## Outputs

- results_table.tsv with per-bin compartment scores, significance calls, flip annotations (A->B, B->A, within-compartment), and HOMER-overlap flags
- Flip fraction tallies over the ~1,981 significant 100 kb bins (~30% A->B, ~44% B->A, ~74% total flips)
- Overlap count of HOMER's 3,042 differential bins with dcHiC calls (expected 1,355 at ±1 bin slack)
- dcHiC standalone IGV HTML visualization of ESC/NPC differential compartments

## How to apply

Run dcHiC's compartment pipeline on the replicate ESC and NPC contact maps at 100 kb resolution: distance normalization (observed/expected) to a correlation matrix, parallelized partial SVD (FBM-backed) for PC1 compartment scores, then quantile normalization of scores across all ESC/NPC maps. For each 100 kb bin, compute the Mahalanobis distance (a multivariate z-score measuring how much the bin is an outlier in compartment-score space), assign chi-square-test p-values, and apply IHW correction under the RefineY_Rconf90_FDR10 settings to call bins significant at FDR < 0.1 (~1,981 bins expected). Compare each significant bin's compartment sign between ESC and NPC: sign change from A to B is an A->B flip, B to A a B->A flip, and same-sign differences are within-compartment changes; tally the flip fractions (~74% flips total: ~30% A->B, ~44% B->A). Load HOMER Differential Scores for the same comparison and intersect HOMER's 3,042 differential bins with dcHiC's significant calls allowing ±1 bin slack, expecting 1,355 overlapping bins. Write per-bin compartment scores, significance calls, flip annotations, and HOMER-overlap flags to results_table.tsv and inspect the dcHiC standalone IGV HTML (e.g., ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html).

## Related tools

- **dcHiC** (Computes PC1 compartment scores via partial SVD, quantile-normalizes them across replicates, and produces Mahalanobis/chi-square/IHW significance calls whose bins are then annotated as A->B, B->A, or within-compartment changes) — https://github.com/ay-lab/dcHiC
- **HOMER** (Provides orthogonal differential compartment scores whose 3,042 ESC/NPC differential bins are intersected with dcHiC's calls (±1 bin slack) to quantify method concordance)
- **IGV** (Underlies dcHiC's standalone HTML browser output used to visualize and sanity-check the per-bin differential compartment and flip annotations)

## Evaluation signals

- Significant-bin count at FDR < 0.1 under RefineY_Rconf90_FDR10 settings is ~1,981 100 kb bins
- Flip fractions among significant bins total ~74%, split ~30% A->B and ~44% B->A
- HOMER concordance check: 1,355 of HOMER's 3,042 differential bins overlap dcHiC calls within ±1 bin slack
- results_table.tsv assigns every significant bin exactly one label from {A->B flip, B->A flip, within-compartment change}, with no unannotated significant bins
- Per-bin PC1 scores are quantile-normalized across all ESC/NPC maps before Mahalanobis distance computation (raw per-map PC1 values would violate comparability)

## Limitations

- Concordance with HOMER is substantial but partial: only 1,355 of HOMER's 3,042 differential bins overlap dcHiC calls even with ±1 bin slack, so flip annotations are method-dependent
- Within-compartment differences (same-sign changes) are biologically significant but easy to overlook if the output is reduced to sign flips only — the article explicitly highlights these
- Single-cell compartment identification is limited by contact-map sparsity; differential calls require pseudo-bulking to at least ~80–100 cells per condition
- Article evaluation is limited to human and mouse Hi-C data; extension to other organisms or Hi-C variants is not demonstrated here
- Nucleolus-associated domain (NAD) enrichment of ESC/NPC differential bins (>10%) was reported only as an initial analysis

## Evidence

- [intro] for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] our results also highlight the importance of generally ignored compartmental differences within the same compartment type (within A or within B: "our results also highlight the importance of generally ignored compartmental differences within the same compartment type (within A or within B"
- [methods] HOMER Differential Scores (for comparison): "HOMER Differential Scores (for comparison)"
- [discussion] Our initial analysis showed that over 10% of all significant compartment differences we found between ESC and NPC belong to nucleolus-associated domains: "Our initial analysis showed that over 10% of all significant compartment differences we found between ESC and NPC belong to nucleolus-associated domains"
- [readme] Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
