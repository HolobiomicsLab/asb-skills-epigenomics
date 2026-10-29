---
name: stouffer-pvalue-combination
description: Use when integrating multiple epigenomic/transcriptomic p-value tracks
  per gene (e.g. expression plus promoter/body/enhancer methylation) into one weighted
  significance score that accounts for non-independence of the combined values.
when_to_use_negative:
- The input is raw counts or signal intensities without per-interval or per-gene p-values
  — SMITE combines p-values, not raw measurements
- Only a single data type is available, so no combination of non-independent p-values
  is needed
- The data types are statistically independent and a simple unweighted Fisher/Stouffer
  combination outside SMITE's correlation-aware framework suffices
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: SMITE
  role: R/Bioconductor package implementing Stouffer combination with correlation
    correction, distance weighting, quantile-permutation normalization, and Fisher-method
    final scoring
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment for running the SMITE pipeline (library(SMITE))
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:e8abee414418ceed54b829efb63c88f8615f668479779965c5f4518edb49a435
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  - igraph
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# stouffer-pvalue-combination

## Summary

Combine correlated component p-values (expression and methylation contexts: promoter, gene body, H3K4me1 enhancer) into a single per-gene significance using SMITE's Stouffer method with distance-based weights and correlation correction, then derive the final Fisher-method score G_i = -2ln(p_i).

## When to use

Use when integrating multiple epigenomic/transcriptomic p-value tracks per gene (e.g. expression plus promoter/body/enhancer methylation) into one weighted significance score that accounts for non-independence of the combined values.

## When NOT to use

- The input is raw counts or signal intensities without per-interval or per-gene p-values — SMITE combines p-values, not raw measurements
- Only a single data type is available, so no combination of non-independent p-values is needed
- The data types are statistically independent and a simple unweighted Fisher/Stouffer combination outside SMITE's correlation-aware framework suffices

## Inputs

- Gene expression table with effect and p-value columns (annotateExpression effect_col=1, pval_col=2)
- Methylation/hydroxymethylation interval BED-like table with a p-value column
- H3K4me1 ChIP-seq peak BED (histone_h3k4me1)
- hg19 gene annotation BED (hg19_genes_bed)
- Pre-computed PvalueAnnotation object (e.g. test_annotation_score_data)

## Outputs

- PvalueAnnotation with combined per-gene p-values and scores
- Per-gene adjusted p-values and G = -2ln(p) statistics (scored_gene_pvalues.tsv)
- Weighted vs unweighted score comparison (stouffer_weight_comparison.csv with R-squared)
- Module-level Fisher p-values (Chi-square, 2k df for k genes)

## How to apply

Load the methylation, expression, hg19 gene BED, and H3K4me1 peak data and build a PvalueAnnotation with makePvalueAnnotation (promoter_upstream_distance=1000, promoter_downstream_distance=1000, other_tss_distance=5000). Before combining, clean p-value columns: replace zeros with the minimum non-zero p-value and remove NA rows. Aggregate interval-level p-values within each context using annotateModification with weight_by_method="Stouffer" and optional distance-based weight_by values; the paper shows weighting choice barely matters (R2 = 0.99 between weighted and unweighted final scores). Assign directionality coefficients via makePvalueObject (e.g. methylation_promoter="decrease", methylation_body="increase", methylation_h3k4me1="bidirectional"), normalize with normalizePval(ref="expression", method="rescale") to remove the number-of-p-values bias, then combine component scores with scorePval/stoufferTest using published weights (expression=0.3, methylation_promoter=0.3, methylation_body=0.1, methylation_h3k4me1=0.3), accounting for correlation between non-independent values. Estimate per-gene p-values by Monte Carlo (MCM) bootstrapping and transform to the final score G_i = -2ln(p_i), which follows approximately a Chi-square distribution with 2 degrees of freedom.

## Related tools

- **SMITE** (R/Bioconductor package implementing Stouffer combination with correlation correction, distance weighting, quantile-permutation normalization, and Fisher-method final scoring) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment for running the SMITE pipeline (library(SMITE)))

## Examples

```
test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3, expression=.3, methylation_body=.1, methylation_h3k4me1=.3))
```

## Evaluation signals

- R-squared between distance-weighted and unweighted final scores is approximately 0.99 (Additional file 1: Figure S2), with nearly identical final modules
- The negative natural log of combined significance shows an increasing trend with number of combined p-values before normalizePval(method="rescale") and no trend afterwards (Fig. 3 pattern)
- Every combined p-value lies in (0,1]
- The distribution of G = -2ln(p) matches a Chi-square distribution with 2 df by visual/quantile check
- Module-level Fisher significance computed as Chi-square with 2k degrees of freedom for k genes

## Limitations

- p-values of exactly zero must be replaced with the minimum non-zero p-value and NA p-values dropped before combination
- Combined significance increases with the number of p-values combined per interval unless the quantile-permutation rescale adjustment is applied
- Weights are user-specified (e.g. 0.3/0.3/0.1/0.3); results are robust to weighting choice (R2 ~ 0.99) but the weighting model choice (SMITE-R vs SMITE-F) still changes module membership substantially (only 94-gene overlap)

## Evidence

- [abstract] SMITE combines p-values accounting for correlation between non-independent values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [methods] Stouffer weighting of methylation annotation: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [methods] Rescale normalization step: "test_annotation <- normalizePval(test_annotation, ref="expression", 
                                 method="rescale")"
- [other] Weighting robustness finding: [evidence span withheld — anchor status: not_found]
- [abstract] Weighting capability: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
