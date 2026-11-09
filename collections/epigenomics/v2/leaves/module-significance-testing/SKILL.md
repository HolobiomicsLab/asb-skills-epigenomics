---
name: module-significance-testing
description: Use when a normalized PvalueAnnotation object holds multiple component
  scores per gene (e.g. expression plus methylation_promoter/body and h3k4me1) and
  you need a single per-gene p-value and G statistic to score network modules.
when_to_use_negative:
- Inputs are raw expression or methylation p-value tables that have not yet been annotated
  (makePvalueAnnotation), normalized (normalizePval, method='rescale'), and assembled
  with makePvalueObject
- Only a single data type is available, so no multi-source p-value combination or
  weighting is required
- Gene p-values were produced by a different method whose null distribution is not
  Chi-square(2 df), invalidating Fisher's 2k df module test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: SMITE
  role: Provides scorePval/stoufferTest for the correlation-aware weighted Stouffer
    combination, MCM gene p-value estimation, and Fisher-transformed module scoring
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment in which the SMITE scoring pipeline is executed
- name: igraph
  role: Supplies the spinglass algorithm used to determine modules whose member gene
    scores are combined into module-level significance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:dc2183608a36757aa82eca5009f7dcd649c3f980a22ea5966c93831c17ad9e69
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# module-significance-testing

## Summary

Derives SMITE's final per-gene significance by combining weighted component p-value scores (expression, methylation_promoter/body, h3k4me1) with a correlation-aware Stouffer test, directionality coefficients, and Monte Carlo (MCM) p-value estimation, then transforms gene p-values via Fisher's method G = -2*ln(p) for module-level testing against a Chi-square distribution.

## When to use

Use when a normalized PvalueAnnotation object holds multiple component scores per gene (e.g. expression plus methylation_promoter/body and h3k4me1) and you need a single per-gene p-value and G statistic to score network modules.

## When NOT to use

- Inputs are raw expression or methylation p-value tables that have not yet been annotated (makePvalueAnnotation), normalized (normalizePval, method='rescale'), and assembled with makePvalueObject
- Only a single data type is available, so no multi-source p-value combination or weighting is required
- Gene p-values were produced by a different method whose null distribution is not Chi-square(2 df), invalidating Fisher's 2k df module test

## Inputs

- Normalized PvalueAnnotation object from SMITE containing component scores (expression, methylation_promoter, methylation_body, methylation_h3k4me1)
- Weight vector (e.g. methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3)
- Effect direction map: methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional'
- Gene-interaction network (e.g. igraph object such as the Reactome.Symbol.Igraph.rda network) for module assignment

## Outputs

- Per-gene combined p-values in (0,1] (MCM-estimated) written to scored_gene_pvalues.tsv
- Per-gene Fisher-transformed statistics G = -2*ln(p), approximately Chi-square(2 df)
- Module-level Fisher significance values (Chi-square with 2k df for k genes)

## How to apply

Load the normalized PvalueAnnotation (e.g. the packaged test_annotation_score_data) and call scorePval/stoufferTest with the published weights methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3, applying directionality coefficients (methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional') so modification-expression relationships opposite to the a priori specification are penalized. The Stouffer combination accounts for correlation between non-independent component values. Estimate each gene's p-value by Monte Carlo random sampling of the combined scores (an FDR-like empirical p-value). Transform each gene p-value to G_i = -2*ln(p_i); this statistic is approximately Chi-square distributed with 2 degrees of freedom, and a module of k genes is tested by Fisher's method as Chi-square with 2k degrees of freedom.

## Related tools

- **SMITE** (Provides scorePval/stoufferTest for the correlation-aware weighted Stouffer combination, MCM gene p-value estimation, and Fisher-transformed module scoring) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment in which the SMITE scoring pipeline is executed)
- **igraph** (Supplies the spinglass algorithm used to determine modules whose member gene scores are combined into module-level significance)

## Examples

```
test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3, methylation_body=.1, expression=.3, methylation_h3k4me1=.3))
```

## Evaluation signals

- Every output per-gene p-value lies strictly in (0,1]
- The distribution of G = -2*ln(p) matches a Chi-square distribution with 2 degrees of freedom by visual/quantile check
- Module-level Fisher p-values are computed as Chi-square with 2k degrees of freedom for a module of k genes
- Directionality is respected: component scores opposite to the specified effect directions (e.g. promoter methylation 'decrease' paired with increased expression) reduce rather than boost gene scores
- The scoring script (scorePval with the 0.3/0.1/0.3/0.3 weight vector) runs to completion and writes scored_gene_pvalues.tsv

## Limitations

- Current genomic experiments are underpowered to detect genomic events comprehensively within a network; per-data-type weighting exists precisely to permit integration of individually under-powered data sets
- The combined Stouffer p-values must account for correlation between non-independent component values; ignoring correlation inflates significance
- Modules can implicate network members for which there is no statistically significant individual evidence, so per-gene significance should not be required for module membership

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [abstract] The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3,: "test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3,"
- [discussion] a functional module identified by SMITE is implicated by the cumulative evidence of varied input data over all of its members: "a functional module identified by SMITE is implicated by the cumulative evidence of varied input data over all of its members"
