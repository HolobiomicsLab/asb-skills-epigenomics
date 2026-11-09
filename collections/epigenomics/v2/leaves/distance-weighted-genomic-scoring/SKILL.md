---
name: distance-weighted-genomic-scoring
description: Use when a SMITE PvalueAnnotation has been built from methylation/expression/ChIP-seq
  p-values and you must decide whether to weight interval contributions by distance
  from the TSS, or must verify that weighted and unweighted aggregated scores agree.
when_to_use_negative:
- When the goal is to compare SMITE against FEM or evaluate module identification
  on a network, rather than to test sensitivity of interval scores to the weight scheme
- When input p-values have not been pre-filtered (zero p-values and NA p-values must
  first be replaced with the minimum non-zero p-value and removed, respectively)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMITE
  role: Provides makePvalueAnnotation, annotateExpression, annotateModification (weight_by_method='Stouffer'),
    and extractModSummary to compute and compare weighted vs. unweighted interval
    scores
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment in which the SMITE workflow is executed and the weighted-vs-unweighted
    regression is performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:4aa09042f3d0bf573a074d9b4556a32be16b7b18f1d55de74935ff6863df9735
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# distance-weighted-genomic-scoring

## Summary

A robustness-check skill for SMITE that re-scores genomic intervals using Stouffer-combined p-values with optional distance-based weights (e.g., distance from transcription start site) and demonstrates, via regression against an unweighted run, that final gene scores are essentially unchanged (R^2 ≈ 0.99).

## When to use

Use when a SMITE PvalueAnnotation has been built from methylation/expression/ChIP-seq p-values and you must decide whether to weight interval contributions by distance from the TSS, or must verify that weighted and unweighted aggregated scores agree.

## When NOT to use

- When the goal is to compare SMITE against FEM or evaluate module identification on a network, rather than to test sensitivity of interval scores to the weight scheme
- When input p-values have not been pre-filtered (zero p-values and NA p-values must first be replaced with the minimum non-zero p-value and removed, respectively)

## Inputs

- methylationdata (SMITE example methylation p-value table)
- hg19_genes_bed (hg19 gene annotation BED file from SMITEdatasets)
- histone_h3k4me1 (H3K4me1 peak data used as other_data in makePvalueAnnotation)
- curated_expressiondata (expression table with effect column 1 and p-value column 2)

## Outputs

- stouffer_weight_comparison.csv containing paired distance-weighted and unweighted final scores plus the regression R^2
- Two extractModSummary data frames of combined p-values (weighted and unweighted runs)
- A regression R-squared value expected to be approximately 0.99

## How to apply

Load the SMITE example methylation dataset (methylationdata), hg19 gene annotation BED (hg19_genes_bed), and H3K4me1 peaks (histone_h3k4me1); replace p-values of zero with the minimum non-zero p-value and drop NA p-values. Build the PvalueAnnotation with makePvalueAnnotation using promoter_upstream_distance=1000, promoter_downstream_distance=1000, and other_tss_distance=5000 for the h3k4me1 track, then attach expression with annotateExpression (effect_col=1, pval_col=2). Run annotateModification twice with mod_type='methylation' and mod_corr=FALSE: once with weight_by_method='Stouffer' and weight_by=c(promoter='distance', body='distance', h3k4me1='distance'), and once unweighted. Extract combined p-values from both runs via extractModSummary, regress the weighted scores on the unweighted scores, and confirm the R^2 reproduces the reported ~0.99 (Additional file 1: Figure S2). Write the paired scores plus the R^2 to stouffer_weight_comparison.csv as the deliverable.

## Related tools

- **SMITE** (Provides makePvalueAnnotation, annotateExpression, annotateModification (weight_by_method='Stouffer'), and extractModSummary to compute and compare weighted vs. unweighted interval scores) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment in which the SMITE workflow is executed and the weighted-vs-unweighted regression is performed)

## Examples

```
test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer", weight_by=c(promoter="distance", body="distance", h3k4me1="distance"))
```

## Evaluation signals

- The R-squared from regressing distance-weighted final scores on unweighted final scores is approximately 0.99, matching Additional file 1: Figure S2 of the paper
- Final modules and their annotations are nearly identical between the weighted and unweighted runs
- No zero p-values remain in the methylation input (all replaced with the minimum non-zero p-value) and no NA p-values remain
- stouffer_weight_comparison.csv exists and contains paired scores for both runs plus the R^2 value
- Both annotateModification calls used mod_corr=FALSE and mod_type='methylation', with the weighted run specifying weight_by=c(promoter='distance', body='distance', h3k4me1='distance')

## Limitations

- The reported robustness (R^2 ≈ 0.99) was demonstrated on the SMITE example methylation/H3K4me1/hg19 dataset; other datasets or weight schemes are not covered by this validation
- SMITE combines p-values accounting for correlation between non-independent values, so this unweighted-vs-weighted check with mod_corr=FALSE does not test behavior under correlated modifications
- Only Stouffer weighting with distance-based weights (promoter, body, h3k4me1) is exercised in this workflow

## Evidence

- [abstract] weighting enables under-powered dataset integration: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] annotateModification Stouffer weighting call: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [methods] makePvalueAnnotation with other_data track: "test_annotation <- makePvalueAnnotation(data=hg19_genes, other_data=list(h3k4me1=h3k4me1),"
- [methods] p-value filtering step: "## Replace zeros with minimum non-zero p-value ##
methylation[,5] <- replace(methylation[,5], methylation[,5]==0,"
