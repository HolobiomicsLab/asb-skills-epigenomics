---
name: qpcr-region-coordinate-lookup
description: 'Use when benchmarking differential ChIP-seq region callers against a
  small qPCR-validated ground truth set: you have genomic coordinates of qPCR-tested
  regions and called-region sets (e.g. BED/GRanges) from histoneHMM, Diffreps, Chipdiff
  and Rseg to classify as Y/FP/FN.'
when_to_use_negative:
- The histone mark under study is a narrow peak-like feature rather than a broad diffuse
  modification like H3K27me3 — peak-calling-style methods are the article's stated
  failure mode
- There is no qPCR-validated region set or comparable published ground-truth table,
  so Y/FP/FN labels cannot be judged
- Calls are for a single-sample modified-region task rather than a two-sample differential
  comparison
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: histoneHMM
  role: Source of the differential region calls being validated (bivariate HMM, default
    posterior cutoff); compiled as an R package
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and where the overlap analysis
    is scripted
  repo: http://histonehmm.molgen.mpg.de
- name: Bioconductor
  role: GenomicRanges overlap operations used to intersect qPCR region coordinates
    with called-region sets
- name: Diffreps
  role: Competing differential caller whose default-threshold regions are intersected
    with the qPCR regions
- name: Chipdiff
  role: Competing caller; detected only 5 of the validated differential regions (higher
    false negative rate)
- name: Rseg
  role: Competing caller; detected only 6 of the validated differential regions (higher
    false negative rate)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:61762242251e4edf5f64e5e2e39ae4d5db34405f222048d81955cdae18e84e28
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# qpcr-region-coordinate-lookup

## Summary

Validates differential histone-modification region calls (e.g. histoneHMM H3K27me3 SHR vs BN) by intersecting the chromosomal coordinates of qPCR-surveyed regions with each caller's output on the rn4 rat genome, labeling each method-region pair as Y, FP or FN to reproduce the published Table 3 benchmark.

## When to use

Use when benchmarking differential ChIP-seq region callers against a small qPCR-validated ground truth set: you have genomic coordinates of qPCR-tested regions and called-region sets (e.g. BED/GRanges) from histoneHMM, Diffreps, Chipdiff and Rseg to classify as Y/FP/FN.

## When NOT to use

- The histone mark under study is a narrow peak-like feature rather than a broad diffuse modification like H3K27me3 — peak-calling-style methods are the article's stated failure mode
- There is no qPCR-validated region set or comparable published ground-truth table, so Y/FP/FN labels cannot be judged
- Calls are for a single-sample modified-region task rather than a two-sample differential comparison

## Inputs

- Chromosomal coordinates (rn4) of the 11 qPCR-surveyed differential H3K27me3 regions (Table 3)
- histoneHMM differential region calls from get.differential.regions at the default posterior-probability cutoff
- Called-region sets from Diffreps, Chipdiff and Rseg for the SHR vs BN comparison (default parameters)
- qPCR confirmation status per region (of 7 regions without SHR deletions, all but 2 were confirmed)

## Outputs

- 11 rows x 4 methods (histoneHMM, Diffreps, Chipdiff, Rseg) matrix of Y/FP/FN labels
- qpcr_region_detection_table.csv

## How to apply

Load the SHR vs BN differential region calls from histoneHMM (obtained with get.differential.regions at its default cutoff on the posterior probability) together with the chromosomal coordinates of the 11 qPCR-surveyed H3K27me3 regions from Table 3, all on the rn4 rat genome assembly. Load the competing callers' regions — Diffreps, Chipdiff and Rseg — run with the default parameters and thresholds recommended by their authors, since these are what an end user would obtain. Intersect the 11 qPCR region coordinates with each method's called-region set using GenomicRanges overlap operations. Label each pair Y (call overlaps a qPCR-supported region), FP (call not supported by the qPCR survey) or FN (qPCR-supported region missed by the method). Assemble the 11 rows x 4 methods label matrix; acceptance requires a cell-by-cell match to published Table 3 across all 44 Y/FP/FN labels.

## Related tools

- **histoneHMM** (Source of the differential region calls being validated (bivariate HMM, default posterior cutoff); compiled as an R package) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and where the overlap analysis is scripted) — http://histonehmm.molgen.mpg.de
- **Bioconductor** (GenomicRanges overlap operations used to intersect qPCR region coordinates with called-region sets)
- **Diffreps** (Competing differential caller whose default-threshold regions are intersected with the qPCR regions)
- **Chipdiff** (Competing caller; detected only 5 of the validated differential regions (higher false negative rate))
- **Rseg** (Competing caller; detected only 6 of the validated differential regions (higher false negative rate))

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); hits <- findOverlaps(qpcr_gr, method_gr)  # GenomicRanges overlap on rn4 coordinates
```

## Evaluation signals

- Reproduced 11x4 Y/FP/FN label matrix matches published Table 3 cell-by-cell on all 44 labels
- Of the 7 candidate regions without SHR deletions, all but 2 are qPCR-confirmed — an expected ratio in the ground truth
- Chipdiff's row shows 5 of the validated differential regions detected and Rseg's 6, with the remainder labeled FN (higher false negative rates than histoneHMM)
- Every method-region pair receives exactly one label from {Y, FP, FN}; no region is left unclassified
- All intersections are computed in rn4 coordinates — any interval on another assembly must be lifted or excluded

## Limitations

- Competing methods were run only with the default parameters and thresholds recommended by their authors; tuned settings were not benchmarked
- Version numbers for Chipdiff, Rseg, Diffreps, Pepr, Sicer, Broadpeak and samtools are not stated, hindering exact reproduction
- The qPCR survey covers only 11 candidate regions, so the label matrix is a small, region-specific benchmark rather than a genome-wide performance estimate
- No changelog was found for the source repository, and histoneHMM has no stated version number or archival DOI in the document

## Evidence

- [discussion] we demonstrated that histoneHMM outperforms competing methods in the detection of differentially modified regions: "we demonstrated that histoneHMM outperforms competing methods in the detection of differentially modified regions"
- [discussion] rn4 alignment basis for all coordinates: "All ChIP-seq reads were aligned to the rat genome rn4"
- [discussion] default thresholds rationale for competitors: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [readme] histoneHMM installation path: "devtools::install_github("matthiasheinig/histoneHMM")"
