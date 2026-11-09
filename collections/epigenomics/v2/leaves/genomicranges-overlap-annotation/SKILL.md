---
name: genomicranges-overlap-annotation
description: Use when a benchmark asks whether independently called differential ChIP-seq
  regions (e.g. from histoneHMM, Diffreps, Chipdiff, Rseg) overlap a set of experimentally
  validated coordinates like qPCR-surveyed regions.
when_to_use_negative:
- The input is already a labeled detection matrix or consensus table with no raw coordinate
  sets to intersect.
- Region sets are in different genome assemblies with no liftover available — rn4
  overlaps are invalid across builds.
- The question concerns per-bin classification quality of the HMM (posterior probabilities)
  rather than set-level overlap with validated regions.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Source of the differential region calls being evaluated via overlap against
    qPCR-validated coordinates
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which GenomicRanges overlap operations and the label-matrix
    assembly are executed
- name: Bioconductor
  role: Provides GenomicRanges for the interval intersection of qPCR coordinates with
    each method's called regions on rn4
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:6192dd405c587ae39f4cc47ee30b02828fd6959bb4bfd06f25e7a5dc42a6a0f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# genomicranges-overlap-annotation

## Summary

Systematic cross-method comparison of genomic region calls by intersecting chromosome coordinates with GenomicRanges overlap operations on a fixed genome assembly (rn4 rat), labeling each method-region pair as Y, FP, or FN to reproduce validation tables such as Table 3 of the histoneHMM paper.

## When to use

Use when a benchmark asks whether independently called differential ChIP-seq regions (e.g. from histoneHMM, Diffreps, Chipdiff, Rseg) overlap a set of experimentally validated coordinates like qPCR-surveyed regions.

## When NOT to use

- The input is already a labeled detection matrix or consensus table with no raw coordinate sets to intersect.
- Region sets are in different genome assemblies with no liftover available — rn4 overlaps are invalid across builds.
- The question concerns per-bin classification quality of the HMM (posterior probabilities) rather than set-level overlap with validated regions.

## Inputs

- histoneHMM differential region calls from get.differential.regions (default posterior probability cutoff)
- Called-region BED/coordinate sets from Diffreps, Chipdiff and Rseg (SHR vs BN comparison, author-recommended defaults)
- Chromosomal coordinates of the 11 qPCR-surveyed regions from Table 3
- rn4 rat genome coordinate system for all sets

## Outputs

- qpcr_region_detection_table.csv: 11 rows x 4 methods (histoneHMM, Diffreps, Chipdiff, Rseg) matrix of Y/FP/FN labels

## How to apply

Load each method's called regions — histoneHMM's differential regions from get.differential.regions with its default cutoff on the posterior probability, plus Diffreps, Chipdiff and Rseg runs with default parameters and thresholds recommended by their authors — alongside the reference coordinates (e.g. the 11 qPCR-surveyed regions of Table 3). Ensure all region sets use the same genome build; here rn4 rat coordinates. For each method, intersect the reference coordinates with the called-region set using GenomicRanges overlap operations (findOverlaps/overlapsAny). Label every method-region pair as Y (method call overlaps the validated region), FP (method calls an unsupported region), or FN (validated region missed by the method), then assemble the full region x method label matrix. Acceptance is a cell-by-cell match to the published reference table across all labels.

## Related tools

- **histoneHMM** (Source of the differential region calls being evaluated via overlap against qPCR-validated coordinates) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which GenomicRanges overlap operations and the label-matrix assembly are executed)
- **Bioconductor** (Provides GenomicRanges for the interval intersection of qPCR coordinates with each method's called regions on rn4)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); hits <- findOverlaps(qpcr_gr, calls_gr)
```

## Evaluation signals

- The reproduced qpcr_region_detection_table.csv matches the published Table 3 cell-by-cell across all 44 Y/FP/FN labels (output_matches_reference).
- Every one of the 11 qPCR-surveyed regions appears exactly once per method column — no duplicate or missing rows in the label matrix.
- All region sets are confirmed to be in rn4 coordinates before intersecting; no cross-assembly overlap errors.
- The histoneHMM calls derive from get.differential.regions with its default posterior probability cutoff, and competitor calls use author-recommended default thresholds.
- Label logic is self-consistent: histoneHMM's 7 non-deletion qPCR-confirmed regions contain at most 2 FN cells, consistent with the paper's finding.

## Limitations

- Chipdiff and Rseg detected only 5 and 6 of the validated differential regions, respectively, indicating higher false negative rates for these methods on broad marks like H3K27me3.
- The article gives no version numbers for Chipdiff, Rseg, Diffreps, so the comparison cannot be exactly pinned to software releases.
- Only 7 of the 11 candidate regions were qPCR-supported (4 had SHR deletions), so FP/FN statistics rest on a small validation set.

## Evidence

- [discussion] All ChIP-seq reads were aligned to the rat genome rn4: "All ChIP-seq reads were aligned to the rat genome rn4"
- [discussion] default parameters for competitors: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [readme] package installation: "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
- [abstract] histoneHMM as R package for Bioconductor workflows: "a fast algorithm written in C++ and compiled as an R package. It runs in the popular R computing environment"
