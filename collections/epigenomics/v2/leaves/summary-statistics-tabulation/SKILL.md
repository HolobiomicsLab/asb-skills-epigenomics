---
name: summary-statistics-tabulation
description: Use when you need to tabulate headline counts and distribution summaries
  (N, provenance split, mean/median sizes, segment counts, copy numbers) from a merged
  bioinformatic catalog, e.g. reproducing eccDNAdb's 1270-eccDNA summary.
when_to_use_negative:
- The catalog has not yet been merged and annotated (WGS calls and curated entries
  need assembly first; this skill tabulates, it does not identify eccDNAs)
- You need identification of new eccDNAs from raw WGS data — that is the AmpliconArchitect-based
  detection workflow, not summary tabulation
- You need eccDNA functional or clinical associations (gene expression from TCGA/CCLE,
  chromatin accessibility) rather than descriptive catalog statistics
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AmpliconArchitect
  role: Produced the WGS-derived eccDNA catalog whose entries feed the size, segment-count,
    and copy-number statistics tabulated here
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:04acf385534af17d235a7aa7e8b612f025bff0998baa965e1008f64b636d0df2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# summary-statistics-tabulation

## Summary

Assembles reproducible descriptive statistics for the eccDNAdb catalog — total eccDNA count split by provenance (WGS vs. literature curation), annotated gene count, size distribution, segment-number distribution, and mean copy number — into a single aligned table. Replaces ad-hoc re-reading of paper figures with one verifiable statistics table.

## When to use

Use when you need to tabulate headline counts and distribution summaries (N, provenance split, mean/median sizes, segment counts, copy numbers) from a merged bioinformatic catalog, e.g. reproducing eccDNAdb's 1270-eccDNA summary.

## When NOT to use

- The catalog has not yet been merged and annotated (WGS calls and curated entries need assembly first; this skill tabulates, it does not identify eccDNAs)
- You need identification of new eccDNAs from raw WGS data — that is the AmpliconArchitect-based detection workflow, not summary tabulation
- You need eccDNA functional or clinical associations (gene expression from TCGA/CCLE, chromatin accessibility) rather than descriptive catalog statistics

## Inputs

- WGS-derived eccDNA catalog from AmpliconArchitect analysis of tumor whole-genome sequencing data (per-eccDNA records with size, segments, copy number)
- Manually curated eccDNA set collected from published literature
- Per-eccDNA segment records and copy-count annotations

## Outputs

- eccdna_catalog_statistics.csv — one row per statistic (total N=1270, WGS/curated split 754/516, 54,901 genes, mean 9.4 Mb / median 282 kb size, 61.26% single-segment fraction, mean copy number 11.51), structured to align with Supplementary Tables S2–S5

## How to apply

Load the WGS-derived eccDNA catalog (identified by AmpliconArchitect from tumor WGS data) and the manually curated set from published literature as separate provenance sets, then sum them: 754 WGS-derived + 516 curated = 1270 eccDNAs across 480 samples of 42 cancers. Annotate genes on the merged catalog and report the total (54,901 eccDNA genes). Compute the size distribution over all entries, reporting mean size (9.4 Mb) and median size (282 kb); note the strong mean/median divergence reflects the heavily right-skewed size distribution, so report both. From the per-eccDNA segment records, compute the fraction of single-segment eccDNAs (61.26%), and compute the mean copy number across all entries (11.51). Emit one row per statistic into a single CSV (eccdna_catalog_statistics.csv) structured to align with Supplementary Tables S2–S5 and the values plotted in Figures 1–2, and validate each value against the reference numbers, flagging any mismatch as failure.

## Related tools

- **AmpliconArchitect** (Produced the WGS-derived eccDNA catalog whose entries feed the size, segment-count, and copy-number statistics tabulated here)

## Evaluation signals

- Total eccDNA count equals 1270, with the provenance split exactly 754 (WGS-derived) + 516 (curated)
- Annotated gene count equals 54,901 eccDNA genes
- Size distribution reproduces mean 9.4 Mb and median 282 kb (mean >> median, confirming right-skewed sizes)
- Single-segment fraction equals 61.26% and mean copy count equals 11.51
- Emitted CSV has one row per statistic and aligns with Supplementary Tables S2–S5 and Figures 1–2; any mismatch with reference values is flagged as failure

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so statistics reflect that single detection method
- Only 1270 catalogued eccDNAs across 42 cancers; sequencing datasets for techniques like CIRCLE-seq and CIDER-Seq are scarce and not represented
- Coverage limited to human cancers (patient tissues, PDXs, and cancer cell lines); more species and diseases planned for future updates

## Evidence

- [abstract] eccDNAdb currently includes 1270 eccDNAs identified in 480 samples of 42 cancers: "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [abstract] Total annotated gene count: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] Detection tool for the WGS-derived set: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data"
- [discussion] Single-method limitation: "will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
- [readme] Script ordering convention in the source repo: "The number notation at the beginning of a script filename indicates the order of an analysis. It is easy to follow the steps to repeat the analyses in our study."
