---
name: copy-number-distribution-analysis
description: Use when a merged eccDNA catalog with per-entry copy-number records must
  be summarized as a distribution statistic (e.g., mean copy count) for a catalog
  table or figure aligned with Supplementary Tables S2–S5.
when_to_use_negative:
- The input is raw WGS data requiring eccDNA detection rather than an already-cataloged
  eccDNA set with per-entry copy numbers (use the AmpliconArchitect identification
  pipeline first).
- The eccDNAs of interest were detected by techniques other than AmpliconArchitect
  (e.g., CIRCLE-seq or CIDER-Seq), which eccDNAdb does not currently cover.
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: AmpliconArchitect
  role: Detects the eccDNA elements from tumor WGS data that form the WGS-derived
    portion (754 eccDNAs) of the catalog whose copy numbers are summarized
- name: eccDNAdb
  role: Source database and analysis scripts (numbered by analysis order) that produce
    and annotate the eccDNA catalog including copy-number records
  repo: github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:037ce1cb598238a7b43df22defec17ae9d700183de159b3012e3d5835f8fecb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# copy-number-distribution-analysis

## Summary

Computes the copy-number distribution across an eccDNA catalog (here the 1270 eccDNAs in eccDNAdb from 480 samples of 42 cancers) and reports summary statistics such as the mean copy number (11.51) as part of the database's profile tables.

## When to use

Use when a merged eccDNA catalog with per-entry copy-number records must be summarized as a distribution statistic (e.g., mean copy count) for a catalog table or figure aligned with Supplementary Tables S2–S5.

## When NOT to use

- The input is raw WGS data requiring eccDNA detection rather than an already-cataloged eccDNA set with per-entry copy numbers (use the AmpliconArchitect identification pipeline first).
- The eccDNAs of interest were detected by techniques other than AmpliconArchitect (e.g., CIRCLE-seq or CIDER-Seq), which eccDNAdb does not currently cover.

## Inputs

- WGS-derived eccDNA catalog (754 eccDNAs identified via AmpliconArchitect on tumor WGS data from 3395 samples of 57 cancers)
- Manually curated eccDNA set from published literature (516 eccDNAs)
- Per-eccDNA copy-number records from the merged eccDNAdb catalog (1270 eccDNAs across 480 samples of 42 cancers)

## Outputs

- Mean copy-count statistic (11.51) over all catalog entries
- eccdna_catalog_statistics.csv with one row per statistic, aligned with Supplementary Tables S2–S5 and Figures 1–2

## How to apply

Load the merged eccDNA catalog combining the WGS-derived set (754 eccDNAs identified by AmpliconArchitect analysis of tumor WGS data from 3395 samples of 57 cancers) and the manually curated literature set (516 eccDNAs). For each catalog entry, read its copy-number record, then aggregate across all 1270 eccDNAs to compute the mean copy count (reported as 11.51 in eccDNAdb). Emit the statistic as one row in a summary table (eccdna_catalog_statistics.csv) structured to align with Supplementary Tables S2–S5 and the distributions plotted in Figures 1–2 alongside size (mean 9.4 Mb, median 282 kb) and segment-count (61.26% single-segment) statistics. Validation requires the emitted value to reproduce the reference statistic exactly, with any mismatch flagged as failure.

## Related tools

- **AmpliconArchitect** (Detects the eccDNA elements from tumor WGS data that form the WGS-derived portion (754 eccDNAs) of the catalog whose copy numbers are summarized)
- **eccDNAdb** (Source database and analysis scripts (numbered by analysis order) that produce and annotate the eccDNA catalog including copy-number records) — github.com/solo7773/eccDNAdb

## Examples

```
python 03_eccdna_catalog_statistics.py --wgs-catalog wgs_eccdnas.tsv --curated-catalog curated_eccdnas.tsv --out eccdna_catalog_statistics.csv
```

## Evaluation signals

- Mean copy count over all 1270 catalog entries equals 11.51 exactly; any deviation is flagged as failure
- The underlying catalog totals reconcile: 754 WGS-derived + 516 literature-curated = 1270 eccDNAs across 480 samples of 42 cancers
- The output CSV contains one row per statistic and aligns with Supplementary Tables S2–S5 and Figures 1–2
- Co-reported statistics are consistent with the same merged catalog: mean size 9.4 Mb, median 282 kb, 61.26% single-segment eccDNAs, 54,901 annotated eccDNA genes

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so copy-number statistics inherit that detection bias
- Catalog coverage is limited to human cancers (patient tissues, patient-derived xenografts, and cancer cell lines); other species and diseases are not covered

## Evidence

- [abstract] eccDNAdb currently includes 1270 eccDNAs identified in 480 samples of 42 cancers: "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [abstract] A total of 54,901 eccDNA genes were annotated and included in the database: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] AmpliconArchitect used for WGS-based eccDNA detection: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data"
- [readme] Repository provides ordered analysis scripts: "The number notation at the beginning of a script filename indicates the order of an analysis."
