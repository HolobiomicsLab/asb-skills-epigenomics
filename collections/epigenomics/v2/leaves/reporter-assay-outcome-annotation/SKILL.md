---
name: reporter-assay-outcome-annotation
description: Use when a set of candidate CRE or OCR coordinates must be functionally
  benchmarked against previously tested enhancers, e.g. intersecting scATAC-seq-derived
  OCRs with VISTA e11.5 transgenic reporter regions and recovering per-region positive/negative
  outcomes.
when_to_use_negative:
- The candidate regions already carry functional validation or the analysis goal is
  de novo motif/state inference rather than benchmarking against tested enhancers
- The reporter-assay reference lacks outcome annotations (regions tested but unclassified),
  making outcome annotation impossible without external information
- The candidate and reference coordinates cannot be brought to a common genome build
  (e.g. liftover fails for a large fraction of regions), so overlaps would be spurious
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: SnapATAC
  role: Produces the ~370,000 candidate cis-regulatory element coordinates from the
    55,592 single-nucleus ATAC-seq profiles of the mouse secondary motor cortex that
    are benchmarked against reporter-assay outcomes
  repo: https://github.com/r3fang/SnapATAC
- name: R 3.4.0
  role: Runtime for the GenomicRanges-based coordinate-overlap and outcome-annotation
    steps (R < 3.6 required because rhdf5 does not work with 3.6)
  repo: https://github.com/r3fang/SnapATAC
- name: GenomicRanges
  role: R/Bioconductor operations used to intersect candidate CRE coordinates with
    VISTA-tested enhancer regions and recover the 256 overlapping reporter regions
- name: SnapTools
  role: Preprocessing module that builds the snap file (cell-by-bin matrices, fragment
    session) from which the atlas CREs are called before reporter-outcome annotation
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:b2d1457422fd30b0c24052cac1dec974db8be85d4fab63937c53bbf5395738d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - GenomicRanges
  - R
  - R 3.4.0
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reporter-assay-outcome-annotation

## Summary

Annotates candidate cis-regulatory elements (e.g. the ~370,000 candidate CREs from the SnapATAC mouse secondary motor cortex atlas) with transgenic reporter-assay outcomes from a reference database such as the VISTA Enhancer Browser, converting coordinate overlaps into empirical functional validation rates (65% reproducible vs 9.7% background).

## When to use

Use when a set of candidate CRE or OCR coordinates must be functionally benchmarked against previously tested enhancers, e.g. intersecting scATAC-seq-derived OCRs with VISTA e11.5 transgenic reporter regions and recovering per-region positive/negative outcomes.

## When NOT to use

- The candidate regions already carry functional validation or the analysis goal is de novo motif/state inference rather than benchmarking against tested enhancers
- The reporter-assay reference lacks outcome annotations (regions tested but unclassified), making outcome annotation impossible without external information
- The candidate and reference coordinates cannot be brought to a common genome build (e.g. liftover fails for a large fraction of regions), so overlaps would be spurious

## Inputs

- BED/coordinate table of candidate cis-regulatory elements (e.g. ~370,000 CREs from the mouse secondary motor cortex SnapATAC atlas, mm10)
- Reporter-assay database table (e.g. VISTA Enhancer Browser tested enhancers with assay outcomes, mm9), including outcome columns for reproducible expression vs no reproducible expression
- Chain files for mm9↔mm10 liftover to harmonize genome builds

## Outputs

- Per-region annotation table of overlapping VISTA-tested regions with reporter-assay outcome (positive/negative) and CRE overlap coordinates
- Summary validation rates file (e.g. vista_cre_validation.csv) containing the reproducible-expression fraction (167/256 ≈ 65%) and the background rate (9.7%) for comparison

## How to apply

Harmonize the candidate CRE BED coordinates and the reporter-assay table (e.g. VISTA Enhancer Browser regions tested in e11.5 mouse embryos) to a common mouse genome build, applying mm9/mm10 liftover where needed. Load both sets as GenomicRanges objects in R and intersect them with coordinate-overlap operations to recover the subset of previously tested reporter regions that overlap candidate CREs (in the atlas analysis this recovered exactly 256 VISTA-tested regions). Annotate each overlapping region with its assay outcome — reproducible expression in at least one embryonic tissue vs no reproducible expression. Compute the positive fraction (167/256 ≈ 65%) and compare it against the background rate for reporter regions lacking canonical enhancer marks (9.7%); write the per-region annotation table plus summary rates (e.g. vista_cre_validation.csv). The rationale is that candidate OCRs validated by this overlap rate support the claim that SnapATAC's unbiased, peak-free clustering recovers functional cis-elements from rare cell types.

## Related tools

- **SnapATAC** (Produces the ~370,000 candidate cis-regulatory element coordinates from the 55,592 single-nucleus ATAC-seq profiles of the mouse secondary motor cortex that are benchmarked against reporter-assay outcomes) — https://github.com/r3fang/SnapATAC
- **R 3.4.0** (Runtime for the GenomicRanges-based coordinate-overlap and outcome-annotation steps (R < 3.6 required because rhdf5 does not work with 3.6)) — https://github.com/r3fang/SnapATAC
- **GenomicRanges** (R/Bioconductor operations used to intersect candidate CRE coordinates with VISTA-tested enhancer regions and recover the 256 overlapping reporter regions)
- **SnapTools** (Preprocessing module that builds the snap file (cell-by-bin matrices, fragment session) from which the atlas CREs are called before reporter-outcome annotation) — https://github.com/r3fang/SnapTools

## Examples

```
library(GenomicRanges); hits <- findOverlaps(vista_gr, cre_gr, minoverlap = 1); positive <- sum(vista_gr$reproducible[unique(queryHits(hits))]); cat(positive, '/', length(vista_gr), '=', positive/length(vista_gr))
```

## Evaluation signals

- Output contains exactly the expected number of overlapping reporter-tested regions (the VISTA comparison recovered exactly 256 regions)
- Computed reproducible-expression fraction equals 167/256 ≈ 65%
- The positive rate (65%) exceeds the 9.7% background rate reported for VISTA regions lacking canonical enhancer marks
- All coordinate comparisons succeed only after both datasets are lifted to a common build (mm9/mm10 liftover completed) — no build-mismatch overlaps
- The per-region table carries an outcome field for every overlapping region (reproducible expression vs no reproducible expression), no missing annotations

## Limitations

- Functional validation of additional open chromatin regions, especially from rare cell types, is still needed; the authors state further experiments are required beyond computational overlap
- Iterative clustering in peak-based pipelines underrepresents peaks in rare populations, so reporter benchmarking built on such CREs may miss functional elements in rare cell types
- Coordinate liftover between mm9 and mm10 can drop regions that fail mapping, biasing the overlap set before outcome annotation

## Evidence

- [abstract] The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [methods] GenomicRanges overlap usage grounded in paper's linking of distal elements: "SnapATAC links distal elements to putative target genes"
- [methods] SnapATAC has two components: "SnapATAC has two components: [Snaptools](https://github.com/r3fang/SnapTools) and [SnapATAC](https://github.com/r3fang/SnapATAC)."
- [readme] README pre-print title supporting functional cis-element recovery claim: "Fast and Accurate Clustering of Single Cell Epigenomes Reveals Cis-Regulatory Elements in Rare Cell Types"
- [readme] R version constraint relevant to running the annotation pipeline: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
