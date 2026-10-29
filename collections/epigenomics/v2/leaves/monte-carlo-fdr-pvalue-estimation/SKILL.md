---
name: monte-carlo-fdr-pvalue-estimation
description: Use when combining non-independent component p-values/scores per gene
  (e.g. expression plus methylation promoter/body/histone data) and an empirical per-gene
  p-value is needed because parametric assumptions fail under correlation.
when_to_use_negative:
- Component p-values at each locus are independent of each other, so a standard parametric
  Stouffer or Fisher combination without correlation correction is valid
- The input is a single data type with no multi-omic component scores to combine —
  scorePval's weighted Stouffer and Monte Carlo machinery has nothing to integrate
- Loci were pruned to only the most significant subset of effects or pairwise intersections,
  the bias-inducing design SMITE was built to avoid
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMITE
  role: Provides scorePval/stoufferTest to run the correlation-aware weighted Stouffer
    combination and Monte Carlo per-gene p-value estimation, plus the PvalueAnnotation
    object that carries component scores
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Execution environment in which the SMITE library is loaded and the scoring
    pipeline is scripted
- name: igraph
  role: Consumes the Monte Carlo-derived per-gene significance scores when building
    modules with the spinglass algorithm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:99d6f1b879905e393eab8969530a4ef87c9ae886de3fdfb0691912a5751021e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  - igraph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# monte-carlo-fdr-pvalue-estimation

## Summary

SMITE's procedure for assigning a per-gene significance value to a weighted Stouffer combination of component scores (expression, methylation_promoter/body/h3k4me1) by Monte Carlo random sampling, then converting the resulting p-value to G = -2ln(p), which approximates a Chi-square distribution with 2 degrees of freedom for downstream module scoring via Fisher's method.

## When to use

Use when combining non-independent component p-values/scores per gene (e.g. expression plus methylation promoter/body/histone data) and an empirical per-gene p-value is needed because parametric assumptions fail under correlation.

## When NOT to use

- Component p-values at each locus are independent of each other, so a standard parametric Stouffer or Fisher combination without correlation correction is valid
- The input is a single data type with no multi-omic component scores to combine — scorePval's weighted Stouffer and Monte Carlo machinery has nothing to integrate
- Loci were pruned to only the most significant subset of effects or pairwise intersections, the bias-inducing design SMITE was built to avoid

## Inputs

- Normalized PvalueAnnotation object (e.g. packaged test_annotation_score_data) containing expression, methylation_promoter, methylation_body, and methylation_h3k4me1 component scores
- Component score weights vector (methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3)
- Directionality specification via effect_directions (methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional')

## Outputs

- Per-gene empirical (Monte Carlo) p-values in (0,1]
- Per-gene Fisher statistics G = -2ln(p) approximating Chi-square(2 df)
- scored_gene_pvalues.tsv with per-gene p-values and G statistics
- Module-level Fisher p-values computed as Chi-square with 2k degrees of freedom

## How to apply

After normalizePval rescaling and applying effect_directions (methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional'), run SMITE's scorePval to combine component scores with Stouffer's method using the published weights (methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3), which accounts for correlation between non-independent values. Estimate each gene's p-value by Monte Carlo random sampling of the combined scores, yielding an FDR-like empirical p-value per gene. Transform each gene p-value to G = -2ln(p); this statistic approximately follows a Chi-square distribution with 2 degrees of freedom, and module-level Fisher significance follows Chi-square with 2k degrees of freedom for k genes. Write per-gene p-values and G statistics to scored_gene_pvalues.tsv for downstream module building with igraph's spinglass algorithm.

## Related tools

- **SMITE** (Provides scorePval/stoufferTest to run the correlation-aware weighted Stouffer combination and Monte Carlo per-gene p-value estimation, plus the PvalueAnnotation object that carries component scores) — https://github.com/GreallyLab/SMITE
- **R** (Execution environment in which the SMITE library is loaded and the scoring pipeline is scripted)
- **igraph** (Consumes the Monte Carlo-derived per-gene significance scores when building modules with the spinglass algorithm)

## Examples

```
test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3))
```

## Evaluation signals

- Every output per-gene p-value lies in the interval (0,1]
- The distribution of G = -2ln(p) statistics matches a Chi-square distribution with 2 degrees of freedom by visual/quantile check
- Module-level Fisher p-values are computed as Chi-square with 2k degrees of freedom for a module of k genes
- The scoring script runs to completion on the packaged pre-scored test_annotation_score_data without errors
- scored_gene_pvalues.tsv contains both p-value and G statistic columns for every annotated gene

## Limitations

- Current genomic experiments are underpowered to detect genomic events comprehensively within a network, so individual component evidence can be weak and weights matter
- Per-gene scores from the Monte Carlo step are FDR-like empirical estimates, not exact parametric p-values
- Non-independence of component p-values requires the correlation-aware combination; naive Fisher/Stouffer on correlated inputs would misstate significance

## Evidence

- [abstract] SMITE combines p-values accounting for correlation between non-independent values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [methods] scorePval call with weights: "test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3,"
- [abstract] Weighting contribution of each data type: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
