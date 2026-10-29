---
name: genomic-blacklist-filtering
description: Use when building a DiffBind workflow on ATAC-seq or ChIP-seq peaks (e.g.,
  mouse/mm10 cKO-vs-WT samples) before counting reads over consensus peaks, to exclude
  ENCODE blacklist artifact regions from downstream DAR generation.
when_to_use_negative:
- The peak data are on a different genome build than the blacklist constant (e.g.,
  hg38 data with DBA_BLACKLIST_MM10) — use the matching blacklist instead
- The input is already a finalized DAR report or BED file; blacklist removal must
  occur upstream, before dba.count and differential analysis
- Working outside DiffBind (e.g., single-cell ArchR sgRNA pipelines), where blacklist
  handling follows a different tool-specific mechanism
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3173
tools:
- name: DiffBind
  role: Provides dba.blacklist, used to remove ENCODE blacklist regions from the dba
    object prior to consensus peak counting and differential analysis
  repo: https://github.com/ZeyuL01/BIT
- name: MACS2
  role: Produces the narrowPeak files that form the peak sets entering the DiffBind
    object to which blacklist filtering is applied
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:a2bdcbf053c5ac8e5deabedbbc156f2f105d06f8e9e9648889bcffeb862963c7
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genomic-blacklist-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes ENCODE blacklist regions from a peak set during differential accessibility analysis in DiffBind, so that artifact-prone genomic loci (e.g., centromeric and high-signal mm10 regions) do not contaminate the differentially accessible region (DAR) set used as input to BIT.

## When to use

Use when building a DiffBind workflow on ATAC-seq or ChIP-seq peaks (e.g., mouse/mm10 cKO-vs-WT samples) before counting reads over consensus peaks, to exclude ENCODE blacklist artifact regions from downstream DAR generation.

## When NOT to use

- The peak data are on a different genome build than the blacklist constant (e.g., hg38 data with DBA_BLACKLIST_MM10) — use the matching blacklist instead
- The input is already a finalized DAR report or BED file; blacklist removal must occur upstream, before dba.count and differential analysis
- Working outside DiffBind (e.g., single-cell ArchR sgRNA pipelines), where blacklist handling follows a different tool-specific mechanism

## Inputs

- DiffBind dba object built from a samplesheet pointing to processed BAM files and MACS2 narrowPeak files (2 WT + 2 cKO replicates)
- Built-in ENCODE blacklist intervals for the reference genome (DBA_BLACKLIST_MM10 for mouse mm10)

## Outputs

- Blacklist-filtered DiffBind dba object whose consensus peak set excludes ENCODE blacklist regions
- Cleaned consensus peak intervals carried forward into dba.count and the final DAR BED export (e.g., mm10_RUNX1.bed)

## How to apply

After constructing the DiffBind object from the samplesheet (e.g., 2 WT and 2 cKO replicates with BAM and MACS2 narrowPeak files), remove blacklist regions by calling dba.blacklist on the dba object with the blacklist argument set to DBA_BLACKLIST_MM10 for mouse data. This step is performed in the consensus-peak stage of the pipeline, after peak loading and before/around dba.count (minOverlap = 2) and dba.normalize (DESeq2, native normalization, background library sizes). The rationale is that ENCODE blacklist regions harbor unstructured, reproducible artifactual signal that would otherwise inflate differential calls and pollute the DAR set (mm10_RUNX1.bed) exported via rtracklayer::export.bed for BIT. Order the filtering before dba.contrast (DBA_CONDITION, minMembers = 2) and dba.analyze (DBA_DESeq2) so the DESeq2 differential report (dba.report, th = 0.1) is computed on blacklist-free intervals.

## Related tools

- **DiffBind** (Provides dba.blacklist, used to remove ENCODE blacklist regions from the dba object prior to consensus peak counting and differential analysis) — https://github.com/ZeyuL01/BIT
- **MACS2** (Produces the narrowPeak files that form the peak sets entering the DiffBind object to which blacklist filtering is applied)

## Examples

```
RUNX1_KO <- dba.blacklist(RUNX1_KO, blacklist = DBA_BLACKLIST_MM10)
```

## Evaluation signals

- The returned object is a valid DiffBind dba object after dba.blacklist with no error, and its consensus peak intervals no longer intersect DBA_BLACKLIST_MM10 intervals
- Subsequent dba.count (minOverlap = 2) and dba.report (th = 0.1) run without error and the report retains Fold, p.value, and FDR columns
- The final exported BED (mm10_RUNX1.bed) contains the expected GRanges of differential regions (e.g., ~15,652 ranges spanning mm10 chromosomes) importable by rtracklayer
- Dar sets split cleanly by Fold sign (positive_fold = Fold > 0; lost regions = Fold < 0) with no intervals overlapping the mm10 blacklist

## Limitations

- Blacklist constants are build-specific (DBA_BLACKLIST_MM10 covers mm10); applying them to other builds or species yields wrong or no filtering
- Blacklist removal addresses only known artifactual loci; other sequencing biases such as GC content bias and PCR amplification bias can still reduce the reliability of the input epigenomic region set

## Evidence

- [full_text] Blacklist removal positioned between consensus peak counting and normalization in the DAR-generating workflow: [evidence span withheld — anchor status: not_found]
