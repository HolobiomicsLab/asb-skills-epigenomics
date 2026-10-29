---
name: differential-accessibility-analysis
description: Use when comparing chromatin accessibility between a TR-perturbed condition
  (e.g., Runx1 cKO) and wild-type controls with replicate BAM and narrowPeak files,
  to produce a DAR BED set as downstream query input.
when_to_use_negative:
- Input is already a peak set or BED file without per-sample read counts and replicates
  — differential testing requires replicate-level count data
- Query regions are single-cell-derived cell-type-specific accessible regions (e.g.,
  from SnapATAC2) rather than bulk perturbation-vs-control contrasts
- The goal is motif-based TR inference rather than producing perturbation ground-truth
  DAR sets
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
tools:
- name: DiffBind
  role: Core R framework for counting reads over consensus peaks, DESeq2-based normalization,
    contrast definition, and differential accessibility analysis
- name: DESeq2
  role: Statistical engine used by dba.analyze for differential testing; also used
    for native normalization with background library sizes
- name: MACS2
  role: Peak caller producing the narrowPeak input files (q-value 0.01) that define
    the consensus peak set
- name: rtracklayer
  role: Exports the DAR GRanges to BED format (e.g., mm10_RUNX1.bed)
- name: samtools
  role: Removes duplicates from sorted BAM files (samtools markdup -r) prior to differential
    analysis
- name: BIT
  role: Downstream consumer of the gained/lost DAR BED sets as query region sets for
    Bayesian TR identification
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:288074849940af162581fbca045c5ac0b583b21c6faf46e8b9e14d1a2678ab89
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
  - DESeq2
  - DiffBind
  - MACS2
  - R
  - rtracklayer
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-accessibility-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a differentially accessible region (DAR) set from ATAC-seq perturbation data using DiffBind with DESeq2 normalization, splitting the output into gained (Fold > 0) and lost (Fold < 0) accessibility regions that serve as ground-truth query inputs for TR identification (e.g., BIT).

## When to use

Use when comparing chromatin accessibility between a TR-perturbed condition (e.g., Runx1 cKO) and wild-type controls with replicate BAM and narrowPeak files, to produce a DAR BED set as downstream query input.

## When NOT to use

- Input is already a peak set or BED file without per-sample read counts and replicates — differential testing requires replicate-level count data
- Query regions are single-cell-derived cell-type-specific accessible regions (e.g., from SnapATAC2) rather than bulk perturbation-vs-control contrasts
- The goal is motif-based TR inference rather than producing perturbation ground-truth DAR sets

## Inputs

- samplesheet (CSV) listing 2 WT and 2 cKO replicates with paths to aligned BAM files and MACS2 narrowPeak files
- processed ATAC-seq BAM files (duplicate-removed, e.g., via samtools markdup)
- MACS2 narrowPeak files called at q-value 0.01

## Outputs

- dba.report differential table (e.g., RUNX1_cKO_report.csv) with Fold, p.value, and FDR columns
- BED file of differentially accessible regions (e.g., mm10_RUNX1.bed) importable by rtracklayer as GRanges
- gained-accessility subset (Fold > 0) and lost-accessibility subset (Fold < 0) of DARs

## How to apply

Build a DiffBind object from a four-sample samplesheet (2 WT and 2 cKO replicates) pointing to processed BAM files and MACS2 narrowPeak calls, count reads over consensus peaks with dba.count (minOverlap = 2), and remove ENCODE mm10 blacklist regions via dba.blacklist (DBA_BLACKLIST_MM10). Normalize with dba.normalize using DESeq2 native normalization (DBA_NORM_NATIVE), background library sizes (DBA_LIBSIZE_BACKGROUND), and background = TRUE to avoid composition bias from global accessibility shifts. Define the condition contrast with dba.contrast (DBA_CONDITION, minMembers = 2) and run dba.analyze with DESeq2, then generate the report at FDR threshold th = 0.1 (dba.report). Finally split the report by sign of the Fold column (Fold > 0 = gained, Fold < 0 = lost) and export each DAR group to BED with rtracklayer::export.bed (e.g., mm10_RUNX1.bed).

## Related tools

- **DiffBind** (Core R framework for counting reads over consensus peaks, DESeq2-based normalization, contrast definition, and differential accessibility analysis)
- **DESeq2** (Statistical engine used by dba.analyze for differential testing; also used for native normalization with background library sizes)
- **MACS2** (Peak caller producing the narrowPeak input files (q-value 0.01) that define the consensus peak set)
- **rtracklayer** (Exports the DAR GRanges to BED format (e.g., mm10_RUNX1.bed))
- **samtools** (Removes duplicates from sorted BAM files (samtools markdup -r) prior to differential analysis)
- **BIT** (Downstream consumer of the gained/lost DAR BED sets as query region sets for Bayesian TR identification) — https://github.com/ZeyuL01/BIT

## Examples

```
RUNX1_KO <- dba.count(RUNX1_KO, minOverlap = 2); RUNX1_KO <- dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10); RUNX1_report_tab <- as.data.frame(dba.report(RUNX1_KO, th = 0.1))
```

## Evaluation signals

- Exported BED file contains the expected GRanges of differential regions (e.g., 15,652 ranges spanning mm10 chromosomes) and is importable by rtracklayer
- dba.report table includes Fold, p.value, and FDR columns
- DARs are split into two groups by sign of Fold: Fold > 0 (gained accessibility) and Fold < 0 (lost accessibility) after perturbation
- Blacklist removal confirmed: no ENCODE DBA_BLACKLIST_MM10 regions remain in the consensus peak set
- Normalization applied with native DESeq2 method and background library sizes, so counts are comparable despite global accessibility shifts

## Limitations

- BIT's (and by extension the DAR set's) reliability depends on NGS data quality; sequencing biases such as GC content bias and PCR amplification bias can reduce reliability of input epigenomic region sets
- Biological complexities of TR regulation — dynamic binding across cell types and time, chromatin structure, and cooperative binding — can reduce downstream method performance
- Comparisons reference supplementary figures not present in the provided text, limiting reproducibility of related benchmarking

## Evidence
