---
name: genomic-interval-intersection
description: Use when a set of predicted regulatory element coordinates (e.g. scATAC-seq
  candidate CREs) needs to be intersected with a reference table of experimentally
  validated enhancers to annotate functional status per region.
when_to_use_negative:
- Input is already a feature matrix or cell-by-peak count table rather than genomic
  interval coordinates — use SnapATAC's cell-clustering/peak-calling pipeline instead
- The reference enhancer table lacks coordinate information or a compatible genome
  build, making interval intersection impossible without additional annotation sources
- The goal is linking distal elements to putative target genes rather than assessing
  overlap with externally validated enhancer assays — use SnapATAC's gene-peak pairing
  method instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: SnapATAC
  role: Source of the ~370,000 candidate cis-regulatory element coordinates from the
    mouse secondary motor cortex scATAC-seq atlas to be intersected with VISTA regions
  repo: https://github.com/r3fang/SnapATAC
- name: GenomicRanges
  role: R package for coordinate-overlap operations between candidate CRE intervals
    and VISTA-tested enhancer regions
- name: R 3.4.0
  role: Runtime environment in which SnapATAC and the GenomicRanges intersection are
    executed (>= 3.4.0 & < 3.6.0 required for rhdf5)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:c448d3bb53a4fa520e72cd52aacee80b66cb32911ed8b9759edcca5f4be1737d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - GenomicRanges
  - R
  - R 3.4.0
  - SnapATAC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genomic-interval-intersection

## Summary

Recover which newly identified open chromatin regions (candidate CREs) from the SnapATAC mouse secondary motor cortex atlas were previously functionally tested, by coordinate-overlap against the VISTA Enhancer Browser table, enabling functional validation of ~370,000 predicted cis-regulatory elements.

## When to use

Use when a set of predicted regulatory element coordinates (e.g. scATAC-seq candidate CREs) needs to be intersected with a reference table of experimentally validated enhancers to annotate functional status per region.

## When NOT to use

- Input is already a feature matrix or cell-by-peak count table rather than genomic interval coordinates — use SnapATAC's cell-clustering/peak-calling pipeline instead
- The reference enhancer table lacks coordinate information or a compatible genome build, making interval intersection impossible without additional annotation sources
- The goal is linking distal elements to putative target genes rather than assessing overlap with externally validated enhancer assays — use SnapATAC's gene-peak pairing method instead

## Inputs

- ~370,000 candidate cis-regulatory element coordinates from the SnapATAC mouse secondary motor cortex atlas (BED-style genomic intervals)
- VISTA Enhancer Browser table of previously tested enhancers with transgenic reporter-assay outcomes (e11.5 mouse embryos)
- Genome build harmonization via mm9/mm10 liftover chain files

## Outputs

- vista_cre_validation.csv containing 256 VISTA-tested overlapping regions annotated with per-region reporter-assay outcome
- Summary rates: reproducible-expression fraction (167/256 = 65%) and background rate comparison (9.7%)

## How to apply

Load the ~370,000 candidate cis-regulatory element coordinates from the SnapATAC mouse secondary motor cortex atlas (55,592 single-nucleus ATAC-seq profiles, 31 cell populations) and the VISTA Enhancer Browser table of previously tested enhancers, harmonizing both to a common mouse genome build (mm9/mm10 via liftover). Intersect candidate CREs with VISTA-tested regions using coordinate-overlap operations in R with GenomicRanges to recover the 256 VISTA-tested regions overlapping candidate CREs. Annotate each overlapping VISTA region with its transgenic reporter-assay outcome (reproducible expression in at least one e11.5 embryonic tissue vs no reproducible expression), then compute the reproducible-expression fraction (167/256 = 65%) and compare it to the 9.7% background rate for VISTA regions lacking canonical enhancer marks. Write the per-region annotation table plus summary rates to vista_cre_validation.csv and verify the output contains exactly 256 regions with the expected 65% positive fraction.

## Related tools

- **SnapATAC** (Source of the ~370,000 candidate cis-regulatory element coordinates from the mouse secondary motor cortex scATAC-seq atlas to be intersected with VISTA regions) — https://github.com/r3fang/SnapATAC
- **GenomicRanges** (R package for coordinate-overlap operations between candidate CRE intervals and VISTA-tested enhancer regions)
- **R 3.4.0** (Runtime environment in which SnapATAC and the GenomicRanges intersection are executed (>= 3.4.0 & < 3.6.0 required for rhdf5))

## Examples

```
ov <- findOverlaps(query = GRanges(vista_seqnames, vista_ranges), subject = GRanges(cre_seqnames, cre_ranges)); vista_cre <- vista_df[unique(queryHits(ov)), ]
```

## Evaluation signals

- Output table vista_cre_validation.csv contains exactly 256 VISTA-tested regions overlapping candidate CREs
- Computed reproducible-expression fraction equals 167/256 ≈ 65%
- 65% positive rate exceeds the 9.7% background rate reported for VISTA regions lacking canonical enhancer marks
- All intervals in both datasets map to a single common mouse genome build after mm9/mm10 liftover harmonization
- Every overlapping VISTA region carries a binary transgenic reporter-assay annotation (reproducible expression vs no reproducible expression)

## Limitations

- Further functional validation of additional open chromatin regions is still needed; the authors state further experiments are required
- Iterative clustering strategies may leave peaks in rare cell populations underrepresented, so candidate CREs from rare cell types may be missing from the interval set
- Coordinate intersection requires harmonizing genome builds (mm9/mm10); liftover errors or build mismatches can drop valid regions

## Evidence

- [abstract] The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [discussion] further functional validation still needed: "While further experiments to thoroughly validate the function of these additional open chromatin regions are needed, the ability for SnapATAC to uncover cis-elements from rare cell types"
- [abstract] SnapATAC applied to mouse secondary motor cortex atlas: "SnapATAC is applied to 55,592 single-nucleus ATAC-seq profiles from the mouse secondary motor cortex"
- [readme] SnapATAC R version requirement: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [methods] SnapATAC links distal elements: "SnapATAC links distal elements to putative target genes"
