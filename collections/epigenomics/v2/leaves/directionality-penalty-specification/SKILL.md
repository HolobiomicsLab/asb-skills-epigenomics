---
name: directionality-penalty-specification
description: Use when integrating expression with promoter/body methylation or histone-mark
  p-values in SMITE and you have prior knowledge of the expected direction of each
  modification–expression relationship, so opposite-direction evidence should be penalized
  rather than treated as concordant.
when_to_use_negative:
- When there is no defensible a priori expectation of direction for a modification–expression
  relationship — declaring 'decrease' or 'increase' without biological grounding will
  penalize true signals.
- When only a single data type is available, since directionality penalties apply
  to relationships between component scores, not to one isolated p-value layer.
- When the analysis goal is direction-agnostic aggregation of all significant loci
  rather than SMITE's directional composite scoring — the article positions SMITE
  against methods that prune to the most significant subset of loci.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SMITE
  role: Provides makePvalueObject/scorePval to apply effect_directions and perform
    correlation-aware Stouffer weighting of component scores
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment in which the SMITE directionality and scoring workflow
    is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:ab6d8b10a0efdc01a236336d23b9905bfb5b5c22e497101f1fc52d9f8155ff48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# directionality-penalty-specification

## Summary

Specifies a priori directionality coefficients for each epigenomic/expression component when SMITE combines per-gene p-values, penalizing modification–expression relationships that run opposite to the expected biology (e.g., promoter methylation 'decrease'). This ensures the Stouffer/Fisher composite gene score reflects mechanistic direction, not just statistical significance.

## When to use

Use when integrating expression with promoter/body methylation or histone-mark p-values in SMITE and you have prior knowledge of the expected direction of each modification–expression relationship, so opposite-direction evidence should be penalized rather than treated as concordant.

## When NOT to use

- When there is no defensible a priori expectation of direction for a modification–expression relationship — declaring 'decrease' or 'increase' without biological grounding will penalize true signals.
- When only a single data type is available, since directionality penalties apply to relationships between component scores, not to one isolated p-value layer.
- When the analysis goal is direction-agnostic aggregation of all significant loci rather than SMITE's directional composite scoring — the article positions SMITE against methods that prune to the most significant subset of loci.

## Inputs

- Normalized PvalueAnnotation object (e.g. packaged test_annotation_score_data) containing component p-values for expression, methylation_promoter, methylation_body, and methylation_h3k4me1
- A priori directionality mapping per component (decrease/increase/bidirectional)
- Component weights (e.g. methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3)

## Outputs

- Per-gene combined Stouffer scores with MCM-estimated p-values
- Per-gene Fisher statistic G_i = -2*ln(p_i) approximating Chi-square(2 df), written e.g. to scored_gene_pvalues.tsv
- Module-level Fisher p-values (Chi-square with 2k degrees of freedom for k genes)

## How to apply

Set directionality coefficients at the makePvalueObject step via effect_directions, e.g. c(methylation_promoter='decrease', methylation_body='increase', methylation_h3k4me1='bidirectional'), encoding the a priori biology that promoter hypermethylation represses expression whereas body/h3k4me1 changes may act either way. After normalizePval (rescaled against the expression reference), SMITE combines component scores by Stouffer's method with published weights (methylation_promoter=0.3, methylation_body=0.1, expression=0.3, methylation_h3k4me1=0.3), accounting for correlation between non-independent values; the directionality specification flips component scores whose effect runs against the declared direction so they add less (or penalize) rather than reinforce. The resulting per-gene combined score is assigned an FDR-like p-value by Monte Carlo random sampling, and the final per-gene statistic is G_i = -2*ln(p_i), approximately Chi-square with 2 degrees of freedom (2k df for modules of k genes). Rationale: unweighted or direction-blind pooling would give equal credit to methylation signals consistent with repression and to signals contradicting it, inflating scores at loci with internally conflicting evidence.

## Related tools

- **SMITE** (Provides makePvalueObject/scorePval to apply effect_directions and perform correlation-aware Stouffer weighting of component scores) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment in which the SMITE directionality and scoring workflow is executed)

## Examples

```
test_annotation <- makePvalueObject(test_annotation, effect_directions=c(methylation_promoter="decrease", methylation_body="increase", methylation_h3k4me1="bidirectional"))
```

## Evaluation signals

- Every per-gene output p-value lies in (0,1]
- The distribution of G = -2*ln(p) matches a Chi-square distribution with 2 degrees of freedom by visual/quantile check
- Module-level Fisher p-values are computed as Chi-square with 2k degrees of freedom for k genes
- Genes whose methylation signal contradicts the specified effect_directions receive lower combined scores than they would under direction-blind pooling
- The script runs to completion on the packaged pre-scored test_annotation_score_data without error

## Limitations

- Directionality coefficients depend entirely on correct a priori specification; mis-specifying a component (e.g. marking h3k4me1 'decrease' instead of 'bidirectional') can systematically penalize genuine signals
- Component scores are non-independent and SMITE must account for correlation; ignoring it distorts the Monte Carlo p-values feeding G_i
- Current genomic experiments are underpowered to detect genomic events comprehensively within a network, so weighting/direction choices materially affect individually under-powered data sets

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [abstract] The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] makePvalueObject with effect_directions: "test_annotation <- makePvalueObject(test_annotation, effect_directions=c(methylation_promoter="decrease","
- [methods] scorePval with component weights: "test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3,"
