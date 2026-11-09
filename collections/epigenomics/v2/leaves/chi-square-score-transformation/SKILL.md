---
name: chi-square-score-transformation
description: Use when a weighted, correlation-corrected per-gene p-value (e.g. from
  SMITE's scorePval/stoufferTest with MCM bootstrapping) must be converted into a
  single comparable score statistic and calibrated against a known null distribution
  for gene- or module-level significance.
when_to_use_negative:
- Component p-values are independent and correlation correction plus MCM bootstrapping
  is unnecessary — plain Fisher's method on raw p-values suffices without SMITE's
  resampling
- Input is a single omics layer with no weighted combination of expression and methylation
  component scores to integrate
- The task is module discovery/topology via spin-glass algorithms (igraph spinglass,
  FEM) rather than per-gene score calibration against a Chi-square null
tools:
- name: SMITE
  role: Provides scorePval for correlation-aware weighted Stouffer combination and
    MCM-based per-gene p-value estimation that is then transformed by Fisher's method
    to the Chi-square(2 df) statistic
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment in which the SMITE PvalueAnnotation is loaded, scored,
    and the −2ln(p) transformation applied
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:d639bb21ca19587edc65d9dd7da1b584604e9e911647c5c3a0406fba7d2d60cc
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
yes_rate: 0.5714285714285714
n_claims: 7
---

# chi-square-score-transformation

## Summary

Transforms each gene's integrated SMITE significance into a single score via Fisher's method, G_i = −2ln(p_i), where p_i is the per-gene p-value from MCM bootstrapping of the correlation-aware Stouffer combination of expression and methylation component scores; G is approximately Chi-square distributed with 2 degrees of freedom.

## When to use

Use when a weighted, correlation-corrected per-gene p-value (e.g. from SMITE's scorePval/stoufferTest with MCM bootstrapping) must be converted into a single comparable score statistic and calibrated against a known null distribution for gene- or module-level significance.

## When NOT to use

- Component p-values are independent and correlation correction plus MCM bootstrapping is unnecessary — plain Fisher's method on raw p-values suffices without SMITE's resampling
- Input is a single omics layer with no weighted combination of expression and methylation component scores to integrate
- The task is module discovery/topology via spin-glass algorithms (igraph spinglass, FEM) rather than per-gene score calibration against a Chi-square null

## Inputs

- Normalized PvalueAnnotation object (e.g. packaged pre-scored test_annotation_score_data) with expression and methylation_promoter/body/h3k4me1 component scores
- Named weight vector for component data types (methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3)
- Effect-direction specification (methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional')
- Interaction network (e.g. REACTOME igraph object) for module-level scoring

## Outputs

- Per-gene p-values in (0,1] from Monte Carlo resampling of the Stouffer-combined score
- Per-gene Fisher statistics G_i = −2ln(p_i), approximately Chi-square(2 df)
- Module-level Fisher p-values evaluated as Chi-square with 2k degrees of freedom (e.g. written to scored_gene_pvalues.tsv)

## How to apply

First combine the component scores (methylation_promoter, methylation_body, expression, methylation_h3k4me1) with Stouffer's method using the published weights (0.3, 0.1, 0.3, 0.3), applying directionality coefficients (methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional') so modification-expression relationships opposing the a priori direction are penalized; SMITE's scorePval does this while accounting for correlation between non-independent values. Estimate each gene's p-value by Monte Carlo random sampling of the combined scores. Then transform each gene's p-value p_i to G_i = −2ln(p_i), which approximates a Chi-square distribution with 2 degrees of freedom. At module level, sum member gene statistics and test against a Chi-square distribution with 2k degrees of freedom for k genes.

## Related tools

- **SMITE** (Provides scorePval for correlation-aware weighted Stouffer combination and MCM-based per-gene p-value estimation that is then transformed by Fisher's method to the Chi-square(2 df) statistic) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment in which the SMITE PvalueAnnotation is loaded, scored, and the −2ln(p) transformation applied)

## Examples

```
test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3, methylation_body=.1, expression=.3, methylation_h3k4me1=.3))
```

## Evaluation signals

- Every output per-gene p-value lies in (0,1]
- The empirical distribution of G = −2ln(p) matches a Chi-square distribution with 2 degrees of freedom by visual or quantile check
- Module-level Fisher p-values are computed as Chi-square with 2k degrees of freedom for a module of k genes
- The scoring script runs to completion on the packaged test_annotation_score_data without errors
- Directionality penalties are reflected: components contradicting the a priori effect directions reduce, not increase, gene significance

## Limitations

- Component p-values are non-independent, so the combined statistic's null distribution must be established by Monte Carlo sampling rather than assuming independence
- Data types with individually under-powered signals need explicit weighting; unweighted combination can dilute genuinely dysregulated genes
- Genes may lack statistically significant individual evidence yet still be implicated only through module membership

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [abstract] The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] scorePval weighting call: "test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3,"
- [methods] effect-direction specification: "test_annotation <- makePvalueObject(test_annotation, effect_directions=c(methylation_promoter="decrease","
- [discussion] module-level implication without individual significance: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
