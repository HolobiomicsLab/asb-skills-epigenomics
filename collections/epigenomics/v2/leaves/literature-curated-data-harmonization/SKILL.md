---
name: literature-curated-data-harmonization
description: Use when building or reproducing an integrated catalog that combines
  pipeline-derived records with literature-curation records and you must reconcile
  provenance and emit aggregate statistics.
when_to_use_negative:
- All eccDNAs in the working set come from a single provenance (e.g. only AmpliconArchitect
  output) and no literature-curation set exists to merge
- The task is to identify new eccDNAs from raw WGS data rather than harmonize and
  summarize an existing catalog
- The input is already a harmonized summary table (e.g. existing Supplementary Tables
  S2-S5) — re-merging would double-count entries
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: AmpliconArchitect
  role: Produces the WGS-derived eccDNA calls that form the computational provenance
    set merged with literature-curated entries
- name: eccDNAdb
  role: Target integrated database and reference catalog (1270 eccDNAs, 480 samples,
    42 cancers) whose statistics this skill reproduces
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:247ba0b73ec3ed23b55964712df4798fde76bd5f4ccb25c30d6ea880a27a1ead
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - AmpliconArchitect
  - eccDNAdb
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# literature-curated-data-harmonization

## Summary

Merges computationally identified elements (WGS/AmpliconArchitect-derived eccDNAs) with manually curated entries extracted from published literature into a single harmonized catalog, then computes unified summary statistics (counts, provenance split, size/segment/copy-number distributions) suitable for tabular reporting.

## When to use

Use when building or reproducing an integrated catalog that combines pipeline-derived records with literature-curation records and you must reconcile provenance and emit aggregate statistics.

## When NOT to use

- All eccDNAs in the working set come from a single provenance (e.g. only AmpliconArchitect output) and no literature-curation set exists to merge
- The task is to identify new eccDNAs from raw WGS data rather than harmonize and summarize an existing catalog
- The input is already a harmonized summary table (e.g. existing Supplementary Tables S2-S5) — re-merging would double-count entries

## Inputs

- WGS-derived eccDNA catalog (AmpliconArchitect output over 3395 tumor samples of 57 cancers, incl. patient tissues, PDXs, and cancer cell lines)
- Manually curated eccDNA set from published literature (516 entries)
- Per-eccDNA segment records and copy-number annotations
- Annotated eccDNA gene list (54,901 eccDNA genes)

## Outputs

- eccdna_catalog_statistics.csv — one row per statistic: total eccDNAs (1270), WGS/curated split (754/516), gene count (54,901), mean size (9.4 Mb), median size (282 kb), single-segment fraction (61.26%), mean copy count (11.51)
- Provenance-tagged merged eccDNA catalog

## How to apply

Load the WGS-derived eccDNA catalog (produced by AmpliconArchitect analysis of tumor WGS data) and the manually curated set collected from published articles as separate provenance-tagged sets. Sum the two sets to obtain the total catalog size and provenance split (754 WGS-derived + 516 curated = 1270 eccDNAs across 480 samples of 42 cancers). From the merged catalog, compute size statistics (mean 9.4 Mb, median 282 kb), the fraction of single-segment eccDNAs from per-eccDNA segment records (61.26%), and the mean copy count (11.51). Annotate gene content (54,901 eccDNA genes) and assemble all values into a one-row-per-statistic table aligned with Supplementary Tables S2-S5 and Figures 1-2, writing the result to eccdna_catalog_statistics.csv. Validate by diffing emitted values against the reference numbers; any mismatch is a failure.

## Related tools

- **AmpliconArchitect** (Produces the WGS-derived eccDNA calls that form the computational provenance set merged with literature-curated entries)
- **eccDNAdb** (Target integrated database and reference catalog (1270 eccDNAs, 480 samples, 42 cancers) whose statistics this skill reproduces) — https://github.com/solo7773/eccDNAdb

## Examples

```
python scripts/summarize_catalog.py --wgs wgs_eccdnas.tsv --curated literature_eccdnas.tsv --genes eccdna_genes.tsv --out eccdna_catalog_statistics.csv
```

## Evaluation signals

- Total catalog size equals 754 + 516 = 1270 eccDNAs; any other sum is a failure
- Provenance split matches 754 WGS-derived / 516 literature-curated
- Annotated gene count equals 54,901 eccDNA genes
- Size distribution reproduces mean 9.4 Mb and median 282 kb; single-segment fraction equals 61.26%; mean copy count equals 11.51
- Output table schema aligns one-row-per-statistic with Supplementary Tables S2-S5 and the values plotted in Figures 1-2

## Limitations

- All computationally derived eccDNAs originate from the AmpliconArchitect pipeline only; calls from other tools (e.g. CIRCLE-seq, CIDER-Seq) are absent because few such datasets are available
- Coverage is restricted to human cancers (patient tissues, PDXs, cancer cell lines); other species and diseases are not represented
- Literature-curated entries depend on what was published and manually extracted, so the curated subset may be less systematic than the WGS-derived subset

## Evidence

- [abstract] Provenance split and totals: "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [abstract] Dual provenance workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Gene annotation total: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] Computational tool used for WGS identification: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [discussion] Curated set purpose: "Meanwhile, manual curation of eccDNAs from published articles was also performed to increase the database coverage."
- [discussion] Pipeline-only limitation: "current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
