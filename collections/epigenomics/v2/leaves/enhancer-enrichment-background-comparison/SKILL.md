---
name: enhancer-enrichment-background-comparison
description: Use when a set of predicted open-chromatin regions or candidate CREs
  needs functional benchmarking against an external enhancer catalog with per-region
  assay outcomes, and you must report an enrichment rate versus a defined background
  rate.
when_to_use_negative:
- The input regions are already experimentally validated enhancers — the skill adds
  no information and the background comparison is meaningless.
- The external assay catalog lacks per-region positive/negative outcome labels, so
  no positive rate can be computed for either foreground or background.
- The goal is cell-type-specific activity assignment rather than aggregate functional
  validation of the CRE catalog as a whole.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: SnapATAC
  role: Produced the candidate cis-regulatory element set (~370,000 CREs across 31
    cell populations in mouse secondary motor cortex) that is being functionally validated
    against VISTA reporter assays
  repo: https://github.com/r3fang/SnapATAC
- name: R
  role: Language for the coordinate-overlap computation and fraction calculation (R
    >= 3.4.0 & < 3.6.0 required for the rhdf5 package used by SnapATAC)
- name: GenomicRanges
  role: R/Bioconductor package used to intersect candidate CREs with VISTA-tested
    regions and recover the 256 overlapping tested enhancers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:5956fccabb561328df580e05e90e92009da9db20b85064620b199cf1ff8db22e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - GenomicRanges
  - R
  - SnapATAC
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# enhancer-enrichment-background-comparison

## Summary

Validates computational candidate cis-regulatory elements (CREs) against orthogonal functional annotations — here, ~370,000 candidate CREs from the SnapATAC mouse secondary motor cortex atlas are intersected with VISTA Enhancer Browser transgenic reporter assays, and the fraction of overlaps with reproducible reporter expression (167/256 = 65%) is compared against a background positive rate (9.7%).

## When to use

Use when a set of predicted open-chromatin regions or candidate CREs needs functional benchmarking against an external enhancer catalog with per-region assay outcomes, and you must report an enrichment rate versus a defined background rate.

## When NOT to use

- The input regions are already experimentally validated enhancers — the skill adds no information and the background comparison is meaningless.
- The external assay catalog lacks per-region positive/negative outcome labels, so no positive rate can be computed for either foreground or background.
- The goal is cell-type-specific activity assignment rather than aggregate functional validation of the CRE catalog as a whole.

## Inputs

- ~370,000 candidate cis-regulatory element coordinates from the SnapATAC mouse secondary motor cortex scATAC-seq atlas (BED-like genomic intervals)
- VISTA Enhancer Browser table of transgenic reporter-assay tested enhancers with per-region outcome (reproducible expression vs no reproducible expression)
- Genome-build liftover chain files (mm9/mm10)

## Outputs

- Per-region annotation table of overlapping VISTA-tested regions with reporter-assay outcomes (vista_cre_validation.csv)
- Computed reproducible-expression fraction of overlapping regions (167/256 ≈ 65%)
- Background positive rate for comparison (9.7% for VISTA regions lacking canonical enhancer marks)

## How to apply

First harmonize the candidate CRE coordinates (~370,000 elements from the SnapATAC mouse secondary motor cortex scATAC-seq atlas) and the external enhancer catalog (VISTA Enhancer Browser, e11.5 mouse embryos) to a common genome build via mm9/mm10 liftover. Next, use coordinate-overlap operations in R with GenomicRanges to recover VISTA-tested regions overlapping candidate CREs — yielding 256 previously tested VISTA regions in this analysis. Annotate each overlapping region with its transgenic reporter outcome (reproducible expression vs no reproducible expression), then compute the positive fraction: 167/256 ≈ 65%. Finally, compare this rate against the background rate reported for VISTA regions lacking canonical enhancer marks (9.7%) and interpret enrichment as evidence that the computationally identified open chromatin regions carry genuine enhancer activity.

## Related tools

- **SnapATAC** (Produced the candidate cis-regulatory element set (~370,000 CREs across 31 cell populations in mouse secondary motor cortex) that is being functionally validated against VISTA reporter assays) — https://github.com/r3fang/SnapATAC
- **R** (Language for the coordinate-overlap computation and fraction calculation (R >= 3.4.0 & < 3.6.0 required for the rhdf5 package used by SnapATAC))
- **GenomicRanges** (R/Bioconductor package used to intersect candidate CREs with VISTA-tested regions and recover the 256 overlapping tested enhancers)

## Examples

```
hits <- findOverlaps(cre_gr, vista_gr); sum(vista_positive[unique(subjectHits(hits))]) / length(unique(subjectHits(hits)))  # 167/256 = 0.65 vs 0.097 background
```

## Evaluation signals

- The overlap output must contain exactly 256 VISTA-tested regions that intersect candidate CREs after genome-build harmonization.
- The computed reproducible-expression fraction must equal 167/256 ≈ 65%.
- The 65% positive rate must exceed the 9.7% background rate for VISTA regions lacking canonical enhancer marks.
- Both coordinate sets must be verified to be on the same genome build (mm9/mm10) after liftover before intersection.
- The output CSV must carry per-region reporter-assay outcome annotations, not just aggregate counts.

## Limitations

- Functional validation of additional open chromatin regions — especially those identified from rare cell types — is still needed; the authors state further experiments are required beyond reporter-assay overlap.
- The comparison depends on the VISTA catalog's coverage: only a subset of the ~370,000 candidate CREs were previously tested in VISTA, so the validation covers 256 regions, not the full atlas.
- Background rate (9.7%) is specific to VISTA regions lacking canonical enhancer marks; reusing this skill with other enhancer catalogs requires a newly defined, matched background.

## Evidence

- [abstract] ~370,000 candidate CREs from the motor cortex atlas: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [discussion] Further functional validation of open chromatin regions is still needed: "While further experiments to thoroughly validate the function of these additional open chromatin regions are needed, the ability for SnapATAC to uncover cis-elements from rare cell types"
- [intro] SnapATAC atlas of mouse secondary motor cortex: "we demonstrate the utility of SnapATAC by building a high-resolution single-cell atlas of the mouse secondary motor cortex. This atlas comprises of ~370,000 candidate cis-regulatory elements across"
- [readme] R version constraints for the SnapATAC analysis environment: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
