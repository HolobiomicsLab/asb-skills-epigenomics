---
name: p-value-distribution-normalization
description: Use when integrating p-values/scores from multiple genomic assays whose
  distributions occupy different ranges — e.g. before SMITE scorePval — and you need
  scores comparable to a reference (expression) without changing within-feature ranking.
when_to_use_negative:
- Input data are raw effect sizes or unannotated assay outputs rather than p-values
  already attached to genes via annotateExpression/annotateModification — normalizePval
  expects a populated PvalueAnnotation.
- You need to change cross-feature ranking or apply non-monotonic calibration; the
  rescale method is monotonic by design and cannot reorder scores within a modification-context
  pairing.
- Only a single data type is present and no cross-type comparability problem exists
  — distribution rescaling against a reference is unnecessary.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0203
tools:
- name: SMITE
  role: Provides normalizePval (logit-transform, rescale to expression reference range,
    back-transform) and plotDensityPval for the before/after density comparison
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Runtime environment for loading the PvalueAnnotation object, running normalization,
    and exporting density plots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:511ebe06085044369211f729c5d55f754844d3385184ca2e3386813ffae15e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  - normalizePval
  - plotDensityPval
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# p-value-distribution-normalization

## Summary

Normalizes component p-value distributions from different data types (expression, methylation promoter/body, H3K4me1) onto a common reference scale using SMITE's logit-transform + rescale pipeline, while preserving the rank order of scores so downstream module detection compares features fairly.

## When to use

Use when integrating p-values/scores from multiple genomic assays whose distributions occupy different ranges — e.g. before SMITE scorePval — and you need scores comparable to a reference (expression) without changing within-feature ranking.

## When NOT to use

- Input data are raw effect sizes or unannotated assay outputs rather than p-values already attached to genes via annotateExpression/annotateModification — normalizePval expects a populated PvalueAnnotation.
- You need to change cross-feature ranking or apply non-monotonic calibration; the rescale method is monotonic by design and cannot reorder scores within a modification-context pairing.
- Only a single data type is present and no cross-type comparability problem exists — distribution rescaling against a reference is unnecessary.

## Inputs

- PvalueAnnotation object with annotated p-values per component (expression, methylation_promoter, methylation_body, methylation_h3k4me1), e.g. from SMITE's test_annotation_score_data
- Reference data type name (ref="expression") defining the target distribution range

## Outputs

- PvalueAnnotation object with normalized (rescaled, back-transformed) p-values/scores per component
- Before/after density comparison panels (e.g. density_plots.png) from plotDensityPval

## How to apply

Load the pre-scored PvalueAnnotation object (e.g. data(test_annotation_score_data) for the T. gondii HFF dataset) and first visualize each component with plotDensityPval(pvalue_annotation, ref="expression") to see how methylation_promoter, methylation_body, and methylation_h3k4me1 distributions deviate from the expression reference. Apply normalizePval with ref="expression" and method="rescale": SMITE logit-transforms the p-values, rescales them to the range of the expression reference, then back-transforms — a monotonic operation that leaves the within-feature ordering untouched. Re-plot the densities with plotDensityPval and verify rank preservation, e.g. Spearman rank correlation of 1 between pre- and post-normalization scores for each modification-context pairing. The normalized scores are then suitable inputs to scorePval with data-type weights (e.g. methylation_promoter=0.3) for module detection.

## Related tools

- **SMITE** (Provides normalizePval (logit-transform, rescale to expression reference range, back-transform) and plotDensityPval for the before/after density comparison) — https://github.com/GreallyLab/SMITE
- **R** (Runtime environment for loading the PvalueAnnotation object, running normalization, and exporting density plots)

## Examples

```
data(test_annotation_score_data); test_annotation <- normalizePval(test_annotation, ref="expression", method="rescale"); plotDensityPval(pvalue_annotation=test_annotation, ref="expression")
```

## Evaluation signals

- Spearman rank correlation of 1 between pre- and post-normalization scores within each modification-context pairing (monotonicity invariant).
- plotDensityPval on normalized scores shows distributions shifted to approximately the expression reference range.
- plotCompareScores (e.g. x_name="expression", y_name="methylation_promoter") shows comparability across data types after normalization.
- Normalized p-values remain valid (in [0,1]) after the back-transformation step.
- Reproduced before/after density panels match the comparison shown in the SMITE vignette.

## Limitations

- Rescaling is anchored to a chosen reference (expression); a poor or noisy reference distribution will define the target range for all other components.
- The monotonic transformation improves cross-type comparability but cannot correct non-monotonic distortions between data types.
- Downstream scores depend on user-chosen weights per data type (e.g. methylation_promoter=0.3 in scorePval), which the normalization step does not set.

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [methods] plotDensityPval before normalization: "plotDensityPval(pvalue_annotation=test_annotation, ref="expression")"
- [methods] normalizePval call with rescale method: "test_annotation <- normalizePval(test_annotation, ref="expression", method="rescale")"
- [abstract] SMITE outperforms FEM: "SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data"
