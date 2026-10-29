---
name: atac-seq-dar-input-preparation
description: Use when starting from aligned ATAC-seq data or a DiffBind contrast and
  you must produce a chromosome-coordinate region BED (e.g., mm10_RUNX1.bed) as the
  query input for BIT or comparable TR-identification methods.
when_to_use_negative:
- The downstream analysis needs a gene set rather than genomic regions — BIT's query
  is a region set (e.g., MACS2-called peaks), not a gene list.
- The input is already a ranked TR table (e.g., mm10_RUNX1_rank_table.csv) — this
  skill operates upstream of BIT inference.
- Single-cell data requiring cell-type-specific regions first — SnapATAC2 is applied
  to generate cell-type-specific accessible regions before TR identification.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MACS2
  role: Peak caller producing the ATAC-seq region coordinates; BIT's input example
    is 'Peaks called by MACS2'
- name: DiffBind (dba.blacklist / dba.count / export.bed)
  role: R framework for blacklist removal (DBA_BLACKLIST_MM10), consensus counting
    (minOverlap = 2), DAR calling, and BED export of the query region set
- name: BIT
  role: Downstream consumer of the prepared DAR BED file; it only needs chromosome
    numbers and coordinates
  repo: https://github.com/ZeyuL01/BIT
- name: samtools markdup
  role: PCR/mark duplicate removal on sorted BAMs prior to peak calling
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:1712c2a8c8e2b6573a9b1cd6b0befe5fe2e89219e2dc591debbea70166a06f0a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BART
  - BIT
  - DiffBind (dba.blacklist / dba.count / export.bed)
  - HOMER
  - I-cisTarget
  - MACS2
  - R
  - WhichTF
  - samtools markdup
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# atac-seq-dar-input-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare differentially accessible region (DAR) BED files from ATAC-seq data so they can serve as the user-provided query region set for BIT's Bayesian TR-identification model — covering MACS2 peak calling (q 0.01), DiffBind blacklist removal/counting, FDR/log2FC DAR filtering, and BED export.

## When to use

Use when starting from aligned ATAC-seq data or a DiffBind contrast and you must produce a chromosome-coordinate region BED (e.g., mm10_RUNX1.bed) as the query input for BIT or comparable TR-identification methods.

## When NOT to use

- The downstream analysis needs a gene set rather than genomic regions — BIT's query is a region set (e.g., MACS2-called peaks), not a gene list.
- The input is already a ranked TR table (e.g., mm10_RUNX1_rank_table.csv) — this skill operates upstream of BIT inference.
- Single-cell data requiring cell-type-specific regions first — SnapATAC2 is applied to generate cell-type-specific accessible regions before TR identification.

## Inputs

- Aligned ATAC-seq BAM files (paired-end)
- DiffBind sample sheet / contrast object (e.g., RUNX1 KO vs control)
- mm10 blacklist (DBA_BLACKLIST_MM10) for blacklist filtering
- DiffBind report table with Fold values and coordinates

## Outputs

- DAR BED file in chromosome/coordinate format (e.g., mm10_RUNX1.bed) ready as BIT query input
- Peak files from MACS2 (q-value 0.01)
- DiffBind marker lists filtered at FDR <= 0.05 & abs(Log2FC) >= 2

## How to apply

Start from aligned BAMs: remove duplicates with 'samtools markdup -r' and call peaks with MACS2 using 'macs2 callpeak -f BAM -g mm ... -B -q 0.01' (mm for mouse, q-value 0.01 threshold). In DiffBind, apply blacklist removal ('dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10)') before counting with 'dba.count(minOverlap = 2)'. Generate the contrast report and separate DARs by direction with 'dplyr::filter(Fold > 0)' versus 'Fold < 0'; for marker-based workflows use the cutoff 'FDR <= 0.05 & abs(Log2FC) >= 2'. Export the chosen DAR set as a BED of chromosome coordinates (e.g., mm10_RUNX1.bed) — BIT requires only chromosome numbers and coordinates, which it converts to fixed-length binary overlap vectors against the TR ChIP-seq reference library. The rationale is that BIT's Bayesian inference quality depends directly on the reliability of the input region set, so sequencing-bias control (duplicate removal, blacklist filtering) is applied upstream.

## Related tools

- **MACS2** (Peak caller producing the ATAC-seq region coordinates; BIT's input example is 'Peaks called by MACS2')
- **DiffBind (dba.blacklist / dba.count / export.bed)** (R framework for blacklist removal (DBA_BLACKLIST_MM10), consensus counting (minOverlap = 2), DAR calling, and BED export of the query region set)
- **BIT** (Downstream consumer of the prepared DAR BED file; it only needs chromosome numbers and coordinates) — https://github.com/ZeyuL01/BIT
- **samtools markdup** (PCR/mark duplicate removal on sorted BAMs prior to peak calling)

## Examples

```
macs2 callpeak -f BAM -g mm -n "${PEAK_DIR}${MIDDLE_DIR}${ATAC_NAME_1}" -B -q 0.01
```

## Evaluation signals

- Peak calls exist for every ATAC sample and were generated with MACS2 at q-value 0.01 (-B -q 0.01).
- Blacklist filtering is verifiably applied in the DiffBind script (dba.blacklist with DBA_BLACKLIST_MM10) before dba.count.
- DAR export separates up- and down-accessible regions by Fold sign (Fold > 0 vs Fold < 0) or applies the marker cutoff FDR <= 0.05 & abs(Log2FC) >= 2.
- The exported BED contains only chromosome numbers and coordinates, the minimum requirement for BIT.
- The resulting BED feeds BIT to produce a rank table with expected columns (TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank) and top-ranked TRs such as Bcl11b, Cbfb, Runx1.

## Limitations

- Sequencing biases such as GC content bias (e.g., on the Illumina platform) or PCR amplification bias can reduce the reliability of the input epigenomic region set.
- BIT's performance depends on NGS data quality; poorly controlled duplicates or blacklist-overlapping peaks propagate into the Bayesian inference.
- Dynamic TR binding across cell types and time, chromatin structure, and cooperative binding can reduce downstream TR-identification performance even with well-prepared DARs.

## Evidence

- [methods] MACS2 q-value threshold and BAM peak calling: "macs2 callpeak -f BAM -g mm -n "${PEAK_DIR}${MIDDLE_DIR}${ATAC_NAME_1}" -B -q 0.01"
- [methods] DiffBind blacklist removal on mm10: "RUNX1_KO <- dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10)"
- [methods] DiffBind DAR marker cutoff: "markerLists<-getMarkers(DiffPeaks,cutOff = "FDR <= 0.05 & abs(Log2FC) >= 2")"
- [results] BIT's minimal region-set input requirement: "BIT only requires a set of epigenomic regions containing the chromosome numbers and coordinates of the regions as input (e.g., Peaks called by MACS2"
- [readme] BIT README: regions derive from epigenomic profiling such as ATAC-seq: [evidence span withheld — anchor status: external_doc]
- [discussion] Sequencing-bias limitation on input region sets: "Multiple types of sequencing biases, such as GC content bias or PCR amplification bias, can potentially reduce the reliability of the input epigenomic region set"
