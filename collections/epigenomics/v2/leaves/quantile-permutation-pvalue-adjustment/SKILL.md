---
name: quantile-permutation-pvalue-adjustment
description: Use when combined significance scores from integrating multiple genomic/epigenomic
  p-value tracks show a spurious increasing trend with the number of component p-values
  per genomic interval, and the trend must be removed before scoring modules.
when_to_use_negative:
- Component p-values are not yet annotated and combined into a PvalueAnnotation (run
  makePvalueAnnotation/annotateExpression/annotateModification/makePvalueObject first)
- The goal is module discovery or network scoring (scorePval/spinglass), not diagnosing
  or removing the number-of-p-values bias in combined significance
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3173
tools:
- name: SMITE
  role: R/Bioconductor package providing normalizePval(method="rescale"), plotDensityPval,
    and the PvalueAnnotation data structure used for the quantile-permutation adjustment
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: execution environment for the SMITE adjustment workflow and density plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:711384a6d1609ec7a42fdcaacb480f7d71468edae680b7c94340596f451711a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# quantile-permutation-pvalue-adjustment

## Summary

SMITE's normalizePval(method="rescale") adjustment removes the bias by which combined significance increases with the number of p-values combined per interval, by logit-transforming and rescaling each score against randomly sampled scores from the same quantile bin (illustrated in Fig. 3 of the SMITE paper).

## When to use

Use when combined significance scores from integrating multiple genomic/epigenomic p-value tracks show a spurious increasing trend with the number of component p-values per genomic interval, and the trend must be removed before scoring modules.

## When NOT to use

- Component p-values are not yet annotated and combined into a PvalueAnnotation (run makePvalueAnnotation/annotateExpression/annotateModification/makePvalueObject first)
- The goal is module discovery or network scoring (scorePval/spinglass), not diagnosing or removing the number-of-p-values bias in combined significance

## Inputs

- PvalueAnnotation object with annotated component p-values (e.g. expression and methylation_promoter/modification tracks), such as test_annotation_score_data
- Reference data type to adjust against (ref="expression")

## Outputs

- Normalized PvalueAnnotation with adjusted (rescaled) combined p-values/scores
- Pre- and post-adjustment density plots of combined p-values versus number of combined p-values (Fig. 3-style comparison)

## How to apply

In R, load the pre-computed PvalueAnnotation (e.g. data(test_annotation_score_data) from SMITEdatasets) and first plot the pre-adjustment density of combined p-values with plotDensityPval(pvalue_annotation=test_annotation, ref="expression") to confirm that combined significance rises with the number of combined p-values per interval. Then run the quantile-permutation adjustment via normalizePval(test_annotation, ref="expression", method="rescale"): scores are logit-transformed, binned by quantile, and each R_ij is benchmarked against 100 randomly sampled scores drawn from the same quantile bin, so intervals are compared only to peers with comparable numbers of component p-values. Re-plot the adjusted p-value density and inspect it side-by-side with the pre-adjustment distribution. Accept the output when the pre-adjustment increasing trend is visible but absent after adjustment, reproducing the qualitative pattern of Fig. 3 of the SMITE paper using the T. gondii HFF data set.

## Related tools

- **SMITE** (R/Bioconductor package providing normalizePval(method="rescale"), plotDensityPval, and the PvalueAnnotation data structure used for the quantile-permutation adjustment) — https://github.com/GreallyLab/SMITE
- **R** (execution environment for the SMITE adjustment workflow and density plots)

## Examples

```
data(test_annotation_score_data); plotDensityPval(pvalue_annotation=test_annotation, ref="expression"); test_annotation <- normalizePval(test_annotation, ref="expression", method="rescale")
```

## Evaluation signals

- Pre-adjustment plotDensityPval output shows combined significance increasing with the number of combined p-values per interval
- After normalizePval(method="rescale"), the plot of -ln(combined significance) versus number of combined p-values no longer shows the increasing trend
- The adjustment ran with ref="expression" and the default 100 random draws per score R_ij from the same quantile bin
- The qualitative pattern reproduces Fig. 3 of the SMITE paper on the T. gondii HFF dataset

## Limitations

- The adjustment is benchmarked against a reference data type (ref="expression"); choice of reference affects the rescaling
- Random sampling within quantile bins introduces stochasticity (100 draws per R_ij), so exact adjusted values may vary between runs

## Evidence

- [methods] plotDensityPval: "plotDensityPval(pvalue_annotation=test_annotation, ref="expression")"
- [methods] normalizePval rescale: "test_annotation <- normalizePval(test_annotation, ref="expression", 
                                 method="rescale")"
- [results] Fig 3 trend finding: [evidence span withheld — anchor status: not_found]
- [other] rescale procedure detail: [evidence span withheld — anchor status: not_found]
