---
name: multi-method-peak-caller-agreement-analysis
description: Use when differential ChIP-seq region calls (e.g. SHR vs BN H3K27me3
  on rn4) need validation against qPCR-confirmed loci and against alternative callers'
  outputs.
when_to_use_negative:
- Peak-like histone marks or transcription factors with narrow punctate peaks, where
  peak-style callers are designed and this comparison's false-negative rationale does
  not apply
- No orthogonal validation data (qPCR regions or functionally annotated loci) exists
  to anchor the Y/FP/FN labels
- Method calls are not on the same genome assembly (e.g. mixing rn4 and other builds)
  — lift-over is required before overlap operations
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Produces the reference differential H3K27me3 region calls (default posterior
    cutoff) that are compared against qPCR regions and other callers
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment for running histoneHMM and the GenomicRanges overlap
    operations
- name: Bioconductor
  role: Source of the GenomicRanges infrastructure used to intersect qPCR coordinates
    with called-region sets
- name: Diffreps
  role: Competing differential caller whose region calls are intersected and labeled
    Y/FP/FN against the qPCR-validated regions
- name: Chipdiff
  role: Competing caller that detected only 5 of the qPCR-validated differential regions,
    illustrating its false negative rate on broad marks
- name: Rseg
  role: Competing caller that detected 6 of the qPCR-validated differential regions
    in the SHR vs BN comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:1befd3ca8746d52768749b515ac3ea26ff8fa3933d972cd72d0e03160e018e2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Chipdiff
  - Diffreps
  - R
  - Rseg
  - histoneHMM
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-method-peak-caller-agreement-analysis

## Summary

Cross-validate differential ChIP-seq region calls by intersecting candidate regions (e.g. histoneHMM calls) with qPCR-validated coordinates and with calls from competing peak callers (Diffreps, Chipdiff, Rseg), building a Y/FP/FN label matrix per method to expose false negatives and false positives.

## When to use

Use when differential ChIP-seq region calls (e.g. SHR vs BN H3K27me3 on rn4) need validation against qPCR-confirmed loci and against alternative callers' outputs.

## When NOT to use

- Peak-like histone marks or transcription factors with narrow punctate peaks, where peak-style callers are designed and this comparison's false-negative rationale does not apply
- No orthogonal validation data (qPCR regions or functionally annotated loci) exists to anchor the Y/FP/FN labels
- Method calls are not on the same genome assembly (e.g. mixing rn4 and other builds) — lift-over is required before overlap operations

## Inputs

- Differential region calls from histoneHMM (get.differential.regions output, default posterior probability cutoff)
- Chromosomal coordinates of qPCR-surveyed regions (Table 3, rn4 coordinates)
- Called regions from Diffreps, Chipdiff and Rseg for the same SHR vs BN comparison (author-recommended defaults)
- Rat genome rn4 coordinate system for all region sets

## Outputs

- qpcr_region_detection_table.csv: rows = qPCR regions, columns = methods (histoneHMM, Diffreps, Chipdiff, Rseg), cells = Y/FP/FN labels
- Per-method false negative / false positive counts relative to qPCR-validated regions

## How to apply

Load the differential region calls from histoneHMM (output of get.differential.regions with its default cutoff on the posterior probability) along with the chromosomal coordinates of the qPCR-surveyed regions (Table 3 of the article, e.g. the 11 candidate H3K27me3 regions between SHR and BN rat strains). Load called regions from the competing methods Diffreps, Chipdiff and Rseg run with the default parameters and thresholds recommended by their authors, since these are the results an end user would obtain. Intersect each qPCR region coordinate with each method's called-region set using GenomicRanges overlap operations on the rn4 rat genome coordinates. Label each method-region pair as Y (call overlaps a qPCR-supported differential region), FP (call not supported by qPCR) or FN (qPCR-supported region missed by the method), then assemble the rows-by-methods label matrix. Note the interpretation from the source study: of 7 regions without SHR deletions all but 2 were qPCR-confirmed, while Chipdiff and Rseg detected only 5 and 6 of the validated regions respectively, indicating higher false negative rates for peak-style callers on broad marks.

## Related tools

- **histoneHMM** (Produces the reference differential H3K27me3 region calls (default posterior cutoff) that are compared against qPCR regions and other callers) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment for running histoneHMM and the GenomicRanges overlap operations)
- **Bioconductor** (Source of the GenomicRanges infrastructure used to intersect qPCR coordinates with called-region sets)
- **Diffreps** (Competing differential caller whose region calls are intersected and labeled Y/FP/FN against the qPCR-validated regions)
- **Chipdiff** (Competing caller that detected only 5 of the qPCR-validated differential regions, illustrating its false negative rate on broad marks)
- **Rseg** (Competing caller that detected 6 of the qPCR-validated differential regions in the SHR vs BN comparison)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Reproduced 11 regions x 4 methods label matrix matches the published Table 3 cell-by-cell (all 44 Y/FP/FN labels identical)
- Count of FN entries per method matches published counts (e.g. Chipdiff FN on 5 of validated regions, Rseg FN on 6)
- All qPCR region coordinates and method calls resolved to the same rn4 assembly before GenomicRanges overlap
- histoneHMM calls were generated with get.differential.regions at its default posterior probability cutoff, not a custom threshold
- Every cell in the output matrix is one of exactly Y, FP, or FN — no NAs or unclassified overlaps

## Limitations

- Competing methods (Chipdiff, Rseg, Diffreps) were run with default parameters and no version numbers were recorded, so reproduction depends on unspecified software versions
- Only 7 of the 11 qPCR regions were interpretable (4 had SHR deletions); qPCR surveys of this size give coarse resolution of false negative rates
- Designed for broad histone modifications such as H3K27me3; the article notes methods searching for peak-like features generate many false positive or false negative calls on such data

## Evidence

- [discussion] Competing caller defaults: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [results] False negative rates of Chipdiff and Rseg: [evidence span withheld — anchor status: not_found]
- [readme] histoneHMM scope on broad marks: "histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3."
- [discussion] Assembly used for coordinates: "All ChIP-seq reads were aligned to the rat genome rn4"
- [intro] Peak-style caller failure mode: "Application of methods that search for peak-like features in such data can generate many false positive or false negative calls."
