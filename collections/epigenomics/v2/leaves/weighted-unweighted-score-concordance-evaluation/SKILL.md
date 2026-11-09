---
name: weighted-unweighted-score-concordance-evaluation
description: Use when a SMITE-style p-value aggregation offers optional per-interval
  weights (e.g., TSS distance) and you must decide whether weighting materially changes
  final gene scores, modules, or annotations.
when_to_use_negative:
- The aggregation method has no optional weighting mechanism, so there is no weighted/unweighted
  pair to compare
- The goal is to optimize or select weights for performance rather than to assess
  sensitivity of final scores to the weighting choice
- Input p-values have not yet been cleaned (NAs and zero p-values), since zero p-values
  break the Stouffer combination and must first be replaced with the minimum non-zero
  p-value
tools:
- name: SMITE
  role: Provides annotateModification with weight_by_method="Stouffer" and extractModSummary
    to produce the weighted and unweighted combined p-value scores being compared
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment for running the SMITE workflow and the weighted-vs-unweighted
    regression (R-squared computation)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:d9acd98c54508f56dc087732ace48174efa2a6a08b8e79164712a95ac30f696f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# weighted-unweighted-score-concordance-evaluation

## Summary

Quantifies how much optional weighting (e.g., distance from the transcription start site for promoter, gene body, and h3k4me1 data) alters SMITE's aggregated interval p-values and final gene scores, by regressing distance-weighted Stouffer-combined scores on unweighted scores and reporting the R-squared of concordance.

## When to use

Use when a SMITE-style p-value aggregation offers optional per-interval weights (e.g., TSS distance) and you must decide whether weighting materially changes final gene scores, modules, or annotations.

## When NOT to use

- The aggregation method has no optional weighting mechanism, so there is no weighted/unweighted pair to compare
- The goal is to optimize or select weights for performance rather than to assess sensitivity of final scores to the weighting choice
- Input p-values have not yet been cleaned (NAs and zero p-values), since zero p-values break the Stouffer combination and must first be replaced with the minimum non-zero p-value

## Inputs

- SMITE methylation p-value table (methylationdata) with p-value column (NAs removed, zeros replaced by minimum non-zero p-value)
- hg19 gene annotation BED (hg19_genes_bed)
- H3K4me1 peak track (histone_h3k4me1)
- Expression data with effect and p-value columns (effect_col=1, pval_col=2)

## Outputs

- Paired weighted-vs-unweighted combined interval p-value scores from extractModSummary
- R-squared of weighted-on-unweighted score regression (expected ~0.99)
- stouffer_weight_comparison.csv containing paired scores and the R-squared value
- Comparison of resulting gene modules and annotations between the two runs

## How to apply

Load the SMITE methylation example data (methylationdata), hg19_genes_bed, and histone_h3k4me1, replacing zero p-values with the minimum non-zero p-value and dropping NA p-values before building the PvalueAnnotation. Build the annotation with makePvalueAnnotation using promoter_upstream_distance=1000, promoter_downstream_distance=1000, and other_tss_distance=5000 for the h3k4me1 track, then attach expression via annotateExpression (effect_col=1, pval_col=2). Run annotateModification twice with mod_type="methylation" and mod_corr=FALSE: once with weight_by_method="Stouffer" and weight_by=c(promoter="distance", body="distance", h3k4me1="distance"), and once unweighted, then extract both combined p-value data frames with extractModSummary. Regress the weighted scores on the unweighted scores and compare the R-squared against the reported ~0.99 (Additional file 1: Figure S2); write the paired scores plus R-squared to stouffer_weight_comparison.csv. An R-squared near 0.99 with nearly identical final modules and annotations indicates the analysis is robust to the choice of weights, supporting use of distance weighting without concern about score distortion.

## Related tools

- **SMITE** (Provides annotateModification with weight_by_method="Stouffer" and extractModSummary to produce the weighted and unweighted combined p-value scores being compared) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment for running the SMITE workflow and the weighted-vs-unweighted regression (R-squared computation))

## Examples

```
test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer", weight_by=c(promoter="distance", body="distance", h3k4me1="distance"), mod_type="methylation", mod_corr=FALSE)
```

## Evaluation signals

- R-squared of the regression of distance-weighted scores on unweighted scores is approximately 0.99, matching Additional file 1: Figure S2 of the paper
- Final modules and annotations from the weighted and unweighted runs are nearly identical
- Zero p-values in the methylation p-value column were replaced with the minimum non-zero p-value before aggregation
- NA rows in the p-value column were removed prior to makePvalueAnnotation
- stouffer_weight_comparison.csv exists and contains the paired scores and the reproduced R-squared value

## Limitations

- The R-squared = 0.99 result is specific to the hg19/methylation/H3K4me1 example; robustness to weighting may not generalize to other datasets or tracks
- The comparison uses mod_corr=FALSE (correlation correction disabled) for the methylation annotation, so conclusions do not cover correlated-modification aggregation
- Contribution weighting of data types (e.g., methylation_promoter=0.3 in scorePval) is a separate weighting axis from per-interval distance weighting and is not evaluated by this concordance check

## Evidence

- [abstract] abstract: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] methods: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
