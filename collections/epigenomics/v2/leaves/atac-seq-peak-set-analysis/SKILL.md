---
name: atac-seq-peak-set-analysis
description: Use when you have a set of ATAC-seq peaks or a DiffBind differential
  accessibility report (e.g. Fold/FDR columns from a TR knockout in mouse thymocytes)
  and need to identify or rank transcriptional regulators from the region coordinates.
when_to_use_negative:
- Input is already a gene-expression matrix or a gene set — BIT requires epigenomic
  region coordinates, and BART is the counterpart for query gene sets
- The query regions are single-cell cell-type-specific calls produced by scBasset/SnapATAC2
  when the goal is sequence modeling or cell clustering rather than TR ranking
- The genome build is not supported by BIT's reference library (e.g. non-human/non-mouse
  genomes; BIT ships 10,140 human and 5,681 mouse TR ChIP-seq datasets)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: BIT
  role: Bayesian hierarchical model that ranks transcriptional regulators from the
    query region set and quantifies uncertainty via 95% credible intervals
  repo: https://github.com/ZeyuL01/BIT
- name: DiffBind
  role: Generates the differential accessibility report and DAR sets from ATAC-seq
    count data (dba.blacklist, dba.count with minOverlap = 2)
- name: MACS2
  role: Peak calling from ATAC-seq BAMs to produce the region sets that feed BIT
- name: rtracklayer
  role: Export of DAR subsets as BED files in R
- name: dplyr
  role: Filtering the DiffBind report by Fold > 0 / Fold < 0 to partition DARs
- name: R
  role: Environment in which the whole peak-set analysis workflow runs; BIT ships
    as an R package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:95d66ad813942c8dee058777192dcc038580c85a0db0695275b8b09641feb645
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
  - BIT
  - DiffBind
  - MACS2
  - R
  - dplyr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# atac-seq-peak-set-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Analyze ATAC-seq peak/DAR sets — from MACS2 peak calling through DiffBind differential accessibility to BIT-based transcriptional regulator identification — on genome mm10, producing ranked TR tables with Bayesian credible intervals.

## When to use

Use when you have a set of ATAC-seq peaks or a DiffBind differential accessibility report (e.g. Fold/FDR columns from a TR knockout in mouse thymocytes) and need to identify or rank transcriptional regulators from the region coordinates.

## When NOT to use

- Input is already a gene-expression matrix or a gene set — BIT requires epigenomic region coordinates, and BART is the counterpart for query gene sets
- The query regions are single-cell cell-type-specific calls produced by scBasset/SnapATAC2 when the goal is sequence modeling or cell clustering rather than TR ranking
- The genome build is not supported by BIT's reference library (e.g. non-human/non-mouse genomes; BIT ships 10,140 human and 5,681 mouse TR ChIP-seq datasets)

## Inputs

- ATAC-seq aligned BAM files (mm10)
- DiffBind differential accessibility report (e.g. RUNX1_cKO_report.csv with Fold and FDR columns)
- Full DAR set as BED (e.g. mm10_RUNX1.bed, 15652 ranges)
- Stranded BED files of increased (Fold > 0) and decreased (Fold < 0) accessibility regions
- Reference genome build specification (genome="mm10")

## Outputs

- MACS2 peak files called at q-value 0.01
- Blacklist-filtered and counted DiffBind object
- Split BED files (RUNX1_KO_increased.bed, RUNX1_KO_decreased.bed) without headers or row names
- BIT rank tables (e.g. mm10_RUNX1_rank_table.csv) with columns Theta_i, lower/upper 95% CI, BIT_score, BIT_score_lower/upper, and Rank

## How to apply

Call peaks from aligned BAMs with MACS2 using `-f BAM -g mm -q 0.01`, remove artifacts with dba.blacklist (DBA_BLACKLIST_MM10), and count with dba.count(minOverlap = 2) to build the differential accessibility report (e.g. RUNX1_cKO_report.csv). Split the DARs by sign of the Fold column using dplyr (Fold > 0 for increased, Fold < 0 for decreased) and export as headerless BED files (RUNX1_KO_increased.bed, RUNX1_KO_decreased.bed) with rtracklayer. Run BIT with genome="mm10" on the full DAR set and each Fold-sign subset; BIT transforms region coordinates to fixed-length binary vectors and uses a Gibbs sampler over a Bayesian hierarchical model referencing 5,681 mouse TR ChIP-seq datasets to compute Theta_i, 95% credible intervals, BIT_score, and Rank per TR. The rationale for splitting is that increased- and decreased-accessibility regions implicate different regulator classes (e.g. Cdk6 vs Runx1) than the combined set (e.g. Bcl11b).

## Related tools

- **BIT** (Bayesian hierarchical model that ranks transcriptional regulators from the query region set and quantifies uncertainty via 95% credible intervals) — https://github.com/ZeyuL01/BIT
- **DiffBind** (Generates the differential accessibility report and DAR sets from ATAC-seq count data (dba.blacklist, dba.count with minOverlap = 2))
- **MACS2** (Peak calling from ATAC-seq BAMs to produce the region sets that feed BIT)
- **rtracklayer** (Export of DAR subsets as BED files in R)
- **dplyr** (Filtering the DiffBind report by Fold > 0 / Fold < 0 to partition DARs)
- **R** (Environment in which the whole peak-set analysis workflow runs; BIT ships as an R package)

## Examples

```
bit(query = "mm10_RUNX1.bed", genome = "mm10", output = "mm10_RUNX1_rank_table.csv")
```

## Evaluation signals

- Rank 1 row of each BIT rank table reproduces the expected TR name and BIT_score to the 8 reported decimal places (e.g. Bcl11b 0.10132149 for the full DAR set, Cdk6 0.09366096 for increased, Runx1 0.06684889 for decreased)
- The reported 95% credible interval (e.g. 0.09860792-0.10371893 for Bcl11b) brackets the point BIT_score estimate
- The exported full DAR set contains the expected number of ranges (15652 for mm10_RUNX1.bed) and the Fold > 0 and Fold < 0 partitions are non-overlapping and jointly cover all non-zero Fold rows
- Rank table schema is complete: Theta_i, lower/upper, BIT_score, BIT_score_lower/upper, and Rank columns present for every TR in the mm10 reference library
- Region sets passed to BIT contain only chromosome numbers and coordinates, as BIT requires no additional peak metadata

## Limitations

- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding among TRs
- Sequencing biases such as GC content bias (e.g. Illumina platform) and PCR amplification bias can reduce the reliability of the input epigenomic region set and binding profiles
- As ChIP-seq experiments grow, the number of parameters BIT infers increases significantly, and analyzing dozens or hundreds of query region sets with the Gibbs sampler becomes time-consuming

## Evidence

- [methods] macs2 callpeak -f BAM -g mm -n "${PEAK_DIR}${MIDDLE_DIR}${ATAC_NAME_1}" -B -q 0.01: "macs2 callpeak -f BAM -g mm -n "${PEAK_DIR}${MIDDLE_DIR}${ATAC_NAME_1}" -B -q 0.01"
- [methods] RUNX1_KO <- dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10): "RUNX1_KO <- dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10)"
- [methods] positive_fold <- RUNX1_report_tab %>%
     dplyr::filter(Fold > 0): "positive_fold <- RUNX1_report_tab %>%
     dplyr::filter(Fold > 0)"
- [results] BIT only requires a set of epigenomic regions containing the chromosome numbers and coordinates of the regions as input (e.g., Peaks called by MACS2: "BIT only requires a set of epigenomic regions containing the chromosome numbers and coordinates of the regions as input (e.g., Peaks called by MACS2"
- [readme] BIT is a novel bioinformatics tool designed to identify transcriptional regulators from a user-provided set of epigenomic regions.: [evidence span withheld — anchor status: not_found]
- [abstract] Leveraging a comprehensive library of TR ChIP-seq data, BIT offers a fully integrated Bayesian approach to assess genome-wide consistency between user-provided epigenomic profiling data: "Leveraging a comprehensive library of TR ChIP-seq data, BIT offers a fully integrated Bayesian approach to assess genome-wide consistency between user-provided epigenomic profiling data"
