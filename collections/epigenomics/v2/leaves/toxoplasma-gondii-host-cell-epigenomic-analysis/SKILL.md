---
name: toxoplasma-gondii-host-cell-epigenomic-analysis
description: Use when SMITE's combined significance scores rise with the number of
  component p-values per interval in the T. gondii HFF data and you need to remove
  that p-value-count bias before module scoring.
when_to_use_negative:
- Input p-values have not yet been annotated and combined — the adjustment presupposes
  a populated PvalueAnnotation with a reference score such as expression
- The goal is module identification or network scoring (scorePval/spinglass) rather
  than correcting the p-value-count bias in combined significance
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: SMITE
  role: Provides normalizePval with method="rescale" (logit-transform plus quantile-permutation
    rescaling, 100 draws per R_ij) and plotDensityPval for diagnostic plotting
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment in which the SMITE package functions are called on the
    T. gondii HFF PvalueAnnotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:dad702e05269dc094426c2fd51d9f1e6fb31a09d9f95ff528a649617d15db02e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# toxoplasma-gondii-host-cell-epigenomic-analysis

## Summary

Applies SMITE's quantile-permutation adjustment (logit-transform plus rescaling via normalizePval with method="rescale") to the Toxoplasma gondii HFF host-cell transcriptomic/epigenomic data so that combined significance no longer increases with the number of p-values combined per genomic interval, reproducing Fig. 3 of the SMITE paper.

## When to use

Use when SMITE's combined significance scores rise with the number of component p-values per interval in the T. gondii HFF data and you need to remove that p-value-count bias before module scoring.

## When NOT to use

- Input p-values have not yet been annotated and combined — the adjustment presupposes a populated PvalueAnnotation with a reference score such as expression
- The goal is module identification or network scoring (scorePval/spinglass) rather than correcting the p-value-count bias in combined significance

## Inputs

- Pre-computed SMITE PvalueAnnotation object (test_annotation) loaded via data(test_annotation_score_data) from the SMITEdatasets package
- Toxoplasma gondii HFF host-cell transcriptomic and epigenomic p-value annotations (expression and methylation_promoter scores)

## Outputs

- Adjusted PvalueAnnotation with quantile-permutation-rescaled combined significance scores
- Side-by-side density plots (pre- vs post-adjustment) of combined p-values vs number of combined p-values, reproducing Fig. 3

## How to apply

In R, load the pre-computed PvalueAnnotation with data(test_annotation_score_data), then plot the pre-adjustment density of combined p-values using plotDensityPval(pvalue_annotation=test_annotation, ref="expression") to confirm the trend that combined significance increases with the number of combined p-values. Run the quantile-permutation adjustment with normalizePval(test_annotation, ref="expression", method="rescale"): each score is logit-transformed and rescaled against scores randomly sampled from the same quantile bin (100 draws per R_ij). Re-plot the adjusted p-value density with plotDensityPval and compare it side-by-side with the pre-adjustment distribution. Accept the output when the number-of-p-values trend is visible before adjustment and absent after adjustment, matching the qualitative pattern of Fig. 3.

## Related tools

- **SMITE** (Provides normalizePval with method="rescale" (logit-transform plus quantile-permutation rescaling, 100 draws per R_ij) and plotDensityPval for diagnostic plotting) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment in which the SMITE package functions are called on the T. gondii HFF PvalueAnnotation)

## Examples

```
data(test_annotation_score_data); test_annotation <- normalizePval(test_annotation, ref="expression", method="rescale"); plotDensityPval(pvalue_annotation=test_annotation, ref="expression")
```

## Evaluation signals

- Pre-adjustment plotDensityPval(ref="expression") shows combined significance increasing with the number of combined p-values
- Post-adjustment plotDensityPval shows the number-of-p-values trend is no longer present
- normalizePval was run with method="rescale" on ref="expression", i.e. 100 quantile-matched random draws per R_ij
- Qualitative pattern of the adjusted density matches Fig. 3 of the SMITE paper
- The returned object is a PvalueAnnotation with adjusted (rescaled) scores replacing raw combined significance

## Limitations

- The adjustment benchmarks against randomly sampled scores within quantile bins, so it corrects the count trend statistically rather than adding biological information
- Diagnosis is qualitative: the acceptance criterion is the visible absence of the trend in the density plot, not a numeric threshold

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene"
- [methods] plotDensityPval call in methods: "plotDensityPval(pvalue_annotation=test_annotation, ref="expression")"
- [methods] normalizePval call in methods: "test_annotation <- normalizePval(test_annotation, ref="expression", 
                                 method="rescale")"
- [abstract] SMITE applied to T. gondii dataset: "We apply SMITE to a complex genomic data set including the epigenomic and transcriptomic effects of Toxoplasma gondii infection on human host cells"
