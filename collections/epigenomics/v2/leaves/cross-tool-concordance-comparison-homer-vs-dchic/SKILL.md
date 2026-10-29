---
name: cross-tool-concordance-comparison-homer-vs-dchic
description: Use when you need to validate a differential compartment caller by comparing
  its significant bin calls against an orthogonal tool (e.g. HOMER) on the same paired
  Hi-C conditions.
when_to_use_negative:
- You only have a single Hi-C map per condition and no orthogonal differential-compartment
  calls to compare against — the concordance step requires both dcHiC and HOMER outputs
  on the same binning
- The maps are single-cell Hi-C without pseudo-bulk aggregation at adequate depth;
  sparsity limits reliable compartment identification
- The comparison involves more than two conditions where time-series clustering or
  multi-map analysis (not a pairwise tool-vs-tool overlap) is the goal
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Produces differential compartment significance calls via Mahalanobis distance,
    chi-square p-values, and IHW correction across paired maps
  repo: https://github.com/ay-lab/dcHiC
- name: HOMER
  role: Provides the orthogonal differential scores used as the comparison baseline
    (3042 significant bins at FDR < 0.05)
- name: IGV
  role: Standalone browser visualization of compartment results and differential calls
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:daf2c5ba80a7c73c1fe636b6717367128b88387ea8159bc0fcbda01d1cc6689c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - HOMER
  - IGV
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Cross-tool concordance comparison: HOMER vs dcHiC differential compartments

## Summary

Benchmark dcHiC's statistically significant differential compartment calls (Mahalanobis distance + IHW, FDR < 0.1) against HOMER's differential scores on the same pairwise mouse ESC vs NPC 100 kb Hi-C comparison, quantifying overlap of significant bins to assess method concordance.

## When to use

Use when you need to validate a differential compartment caller by comparing its significant bin calls against an orthogonal tool (e.g. HOMER) on the same paired Hi-C conditions.

## When NOT to use

- You only have a single Hi-C map per condition and no orthogonal differential-compartment calls to compare against — the concordance step requires both dcHiC and HOMER outputs on the same binning
- The maps are single-cell Hi-C without pseudo-bulk aggregation at adequate depth; sparsity limits reliable compartment identification
- The comparison involves more than two conditions where time-series clustering or multi-map analysis (not a pairwise tool-vs-tool overlap) is the goal

## Inputs

- ESC and NPC Hi-C contact maps at 100 kb resolution (including replicate maps per condition)
- HOMER differential scores for the pairwise ESC vs NPC comparison (3042 significant bins at FDR < 0.05)
- dcHiC compartment scores and significance calls (RefineY_Rconf90_FDR10 settings, FDR < 0.1)

## Outputs

- Count of dcHiC significant 100 kb bins at FDR < 0.1 (~1981 expected)
- Compartment flip annotations and fractions (A->B ~30%, B->A ~44%, ~74% flips total)
- Overlap count of HOMER's 3042 differential bins with dcHiC calls at ±1 bin slack (expect 1355)
- results_table.tsv with per-bin compartment scores, significance calls, flip annotations, and HOMER-overlap flags

## How to apply

Run dcHiC's compartment pipeline on 100 kb ESC and NPC contact maps (including replicates): distance normalization to a correlation matrix, parallelized partial SVD for PC1 scores, quantile normalization across all maps, then Mahalanobis-distance significance testing with chi-square p-values and IHW correction under RefineY_Rconf90_FDR10 settings (FDR < 0.1). Annotate each significant bin as A->B flip, B->A flip, or within-compartment change. Separately load HOMER differential scores for the same ESC/NPC comparison and, using ±1 bin slack, compute how many of HOMER's 3042 FDR < 0.05 differential 100 kb bins overlap dcHiC's calls. Report the overlapping count (expect 1355) and flip fractions (~74% total: ~30% A->B, ~44% B->A) as the concordance summary, writing per-bin scores, significance flags, flip annotations, and HOMER-overlap flags to a results table.

## Related tools

- **dcHiC** (Produces differential compartment significance calls via Mahalanobis distance, chi-square p-values, and IHW correction across paired maps) — https://github.com/ay-lab/dcHiC
- **HOMER** (Provides the orthogonal differential scores used as the comparison baseline (3042 significant bins at FDR < 0.05))
- **IGV** (Standalone browser visualization of compartment results and differential calls) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- dcHiC reports ~1981 significant 100 kb bins at FDR < 0.1 for the ESC vs NPC comparison
- Flip fractions total ~74%, decomposing into ~30% A->B and ~44% B->A
- Exactly 1355 of HOMER's 3042 differential bins (FDR < 0.05) overlap dcHiC's calls under ±1 bin slack
- results_table.tsv contains per-bin compartment scores, significance calls, flip annotations, and HOMER-overlap flags with consistent bin coordinates
- pcOri HTML visualization generated under RefineY_Rconf90_FDR10 naming (e.g. ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html) reflects the settings used for the calls

## Limitations

- Concordance is substantial but partial (1355/3042 HOMER bins), reflecting methodological differences between HOMER's differential scores and dcHiC's Mahalanobis-distance framework; the tools should not be treated as interchangeable
- dcHiC uses a different significance regime (FDR < 0.1 with IHW) than HOMER (FDR < 0.05), so overlap statistics depend on the slack window and threshold choices
- Single-cell compartment identification remains limited by contact-map sparsity, even at coarse resolution

## Evidence

- [methods] HOMER provides the orthogonal differential scores used as the comparison baseline: "HOMER Differential Scores (for comparison)"
- [intro] dcHiC's significance pipeline uses Mahalanobis distance, chi-square test, and IHW/FDR correction per bin: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [intro] Comparable compartment scores across maps are obtained via quantile normalization before differential testing: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [readme] dcHiC is the tool whose significance calls are benchmarked against HOMER: "dcHiC is a tool for differential compartment analysis of Hi-C datasets."
- [methods] The ESC/NPC 100 kb analysis settings appear in the visualization file naming: "ESC_NPC_100Kb.RefineY_Rconf90_FDR10.pcOri.html"
