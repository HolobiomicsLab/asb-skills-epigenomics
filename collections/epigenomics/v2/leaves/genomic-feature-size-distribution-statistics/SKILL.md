---
name: genomic-feature-size-distribution-statistics
description: Use when a catalog of genomic features (e.g. eccDNAs from WGS analysis
  plus literature curation) must be reduced to headline distribution statistics —
  total/provenance counts, mean/median feature size, per-feature segment counts, and
  copy numbers — for reproducibility checks.
when_to_use_negative:
- The goal is eccDNA function or clinical-relevance analysis (gene expression, chromatin
  accessibility integration) rather than catalog-level descriptive statistics
- The eccDNA catalog must be re-identified from raw WGS data — use the AmpliconArchitect
  detection pipeline instead of the statistics summarization step
- EccDNAs from non-human species or non-cancer diseases, which are out of scope for
  eccDNAdb
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: AmpliconArchitect
  role: Detects eccDNA elements from WGS data, producing the WGS-derived portion of
    the catalog whose statistics this skill summarizes
  repo: https://github.com/solo7773/eccDNAdb
- name: eccDNAdb
  role: Source database whose 1270 eccDNAs and 54,901 annotated genes are the inputs
    being summarized
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:89bfef59758f9492930ed7d84057adcf7beeff6fbb3fe8d52bb84d69471ea25f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - AmpliconArchitect
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genomic-feature-size-distribution-statistics

## Summary

Summarize a catalog of genomic features (here, eccDNAs from the eccDNAdb database) into a single statistics table reporting catalog provenance counts, size distribution, segment-number distribution, and copy-number distribution, validated against published reference values.

## When to use

Use when a catalog of genomic features (e.g. eccDNAs from WGS analysis plus literature curation) must be reduced to headline distribution statistics — total/provenance counts, mean/median feature size, per-feature segment counts, and copy numbers — for reproducibility checks.

## When NOT to use

- The goal is eccDNA function or clinical-relevance analysis (gene expression, chromatin accessibility integration) rather than catalog-level descriptive statistics
- The eccDNA catalog must be re-identified from raw WGS data — use the AmpliconArchitect detection pipeline instead of the statistics summarization step
- EccDNAs from non-human species or non-cancer diseases, which are out of scope for eccDNAdb

## Inputs

- WGS-derived eccDNA catalog from AmpliconArchitect analysis of 3395 tumor samples (patient tissues, patient-derived xenografts, cancer cell lines)
- Manually curated eccDNA set collected from published literature
- Per-eccDNA segment records (segment number per eccDNA)
- Per-eccDNA copy-number records

## Outputs

- eccdna_catalog_statistics.csv — one row per statistic: total eccDNAs (1270), WGS-derived vs curated split (754/516), annotated gene count (54,901), mean size (9.4 Mb), median size (282 kb), single-segment fraction (61.26%), mean copy count (11.51)

## How to apply

Load the WGS-derived eccDNA catalog (produced by AmpliconArchitect analysis of tumor WGS data) and the manually curated set from published literature as separate provenance sets, and sum them to get the catalog size (754 WGS-derived + 516 curated = 1270 eccDNAs across 480 samples of 42 cancers) and the annotated gene count (54,901 eccDNA genes). Compute the size distribution over the merged catalog, reporting both mean (9.4 Mb) and median (282 kb) — the large gap between mean and median reflects the heavily right-skewed size distribution of eccDNAs, so both statistics are required. From the per-eccDNA segment records, compute the fraction of single-segment eccDNAs (61.26%). Compute the copy-number distribution over all catalog entries, reporting the mean copy count (11.51). Emit one row per statistic into a single table (eccdna_catalog_statistics.csv) structured to align with Supplementary Tables S2-S5 and Figures 1-2, and flag any mismatch against the reference values as a failure.

## Related tools

- **AmpliconArchitect** (Detects eccDNA elements from WGS data, producing the WGS-derived portion of the catalog whose statistics this skill summarizes) — https://github.com/solo7773/eccDNAdb
- **eccDNAdb** (Source database whose 1270 eccDNAs and 54,901 annotated genes are the inputs being summarized) — https://github.com/solo7773/eccDNAdb

## Examples

```
import pandas as pd; df = pd.concat([wgs_df, curated_df]); stats = {'total_eccdnas': len(df), 'mean_size_mb': df['size'].mean()/1e6, 'median_size_kb': df['size'].median()/1e3}; stats.to_csv('eccdna_catalog_statistics.csv')
```

## Evaluation signals

- Total catalog size equals 754 + 516 = 1270 eccDNAs, split between WGS identification and manual literature curation
- Annotated gene count equals 54,901 eccDNA genes
- Size distribution reproduces mean 9.4 Mb and median 282 kb exactly; any mismatch is a failure
- Single-segment fraction equals 61.26% and mean copy count equals 11.51 across all catalog entries
- Output table has one row per statistic and aligns with Supplementary Tables S2-S5 and Figures 1-2

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so statistics reflect a single detection method
- Statistics describe eccDNAs from human cancers only (480 samples of 42 cancers); more species and diseases are planned for future updates
- Few sequencing datasets are available for CIRCLE-seq and CIDER-Seq, so these techniques are not represented in the catalog statistics

## Evidence

- [abstract] eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers): "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [abstract] Total annotated genes: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] WGS identification method: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [readme] Script ordering convention: "The number notation at the beginning of a script filename indicates the order of an analysis."
