---
name: eccdna-catalog-aggregation
description: Use when reproducing eccDNAdb summary statistics or summarizing a merged
  eccDNA catalog into a stats table (counts, size distribution, segment counts, copy
  numbers) with exact reference values.
when_to_use_negative:
- The input is raw WGS data and the goal is eccDNA detection — run AmpliconArchitect
  first rather than aggregating statistics
- The eccDNAs come from only one provenance (e.g. only literature curation) and you
  cannot reconstruct the 754/516 WGS/curated split
- The goal is integration of gene expression, clinical, or ATAC-seq chromatin accessibility
  data rather than catalog-level summary statistics
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: AmpliconArchitect
  role: Tool that produced the WGS-derived portion of the eccDNA catalog; its output
    eccDNA records are one of the two inputs to the aggregation and all summary statistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:d63966310fc5147621d6683bd2328065a7e7d8637a5fdea46de496147cac78e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# eccdna-catalog-aggregation

## Summary

Aggregate the WGS-derived eccDNA catalog (AmpliconArchitect on tumor WGS data) with the manually curated literature set into a single eccDNAdb catalog and compute its summary statistics — total count, provenance split, gene count, size, segment-number, and copy-number distributions.

## When to use

Use when reproducing eccDNAdb summary statistics or summarizing a merged eccDNA catalog into a stats table (counts, size distribution, segment counts, copy numbers) with exact reference values.

## When NOT to use

- The input is raw WGS data and the goal is eccDNA detection — run AmpliconArchitect first rather than aggregating statistics
- The eccDNAs come from only one provenance (e.g. only literature curation) and you cannot reconstruct the 754/516 WGS/curated split
- The goal is integration of gene expression, clinical, or ATAC-seq chromatin accessibility data rather than catalog-level summary statistics

## Inputs

- WGS-derived eccDNA catalog produced by AmpliconArchitect analysis of tumor WGS data
- Manually curated eccDNA set from published literature
- Per-eccDNA segment records (segment counts per eccDNA)
- Per-eccDNA copy-number records

## Outputs

- eccdna_catalog_statistics.csv with one row per statistic (total count, provenance split, gene count, mean/median size, single-segment fraction, mean copy number)

## How to apply

Load the WGS-derived eccDNA set (identified by AmpliconArchitect analysis of tumor whole-genome sequencing data downloaded from public repositories) and the manually curated set from published articles, then union them with provenance labels. Sum the two provenance sets to obtain the total (754 WGS-derived + 516 curated = 1270 eccDNAs across 480 samples of 42 cancers) and count the annotated eccDNA genes (54,901). From the merged catalog compute the size distribution (mean 9.4 Mb, median 282 kb), the per-eccDNA segment-number distribution (61.26% single-segment eccDNAs), and the copy-count distribution (mean 11.51). Emit one row per statistic, structured to align with Supplementary Tables S2–S5 and the values plotted in Figures 1–2, and write to eccdna_catalog_statistics.csv. Validation requires exact reproduction of the reference values; any mismatch is flagged as failure.

## Related tools

- **AmpliconArchitect** (Tool that produced the WGS-derived portion of the eccDNA catalog; its output eccDNA records are one of the two inputs to the aggregation and all summary statistics)

## Evaluation signals

- Total catalog count equals exactly 1270 eccDNAs (754 WGS-derived + 516 curated)
- Annotated gene count equals exactly 54,901 eccDNA genes
- Size distribution reports mean 9.4 Mb and median 282 kb
- Segment-number distribution reports 61.26% single-segment eccDNAs
- Copy-count distribution reports mean copy number 11.51; any mismatch with these reference values is a failure

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so the catalog inherits that method's biases
- Coverage is restricted to human cancers (patient tissues, PDX, cell lines); more species and diseases are planned only for future updates
- The catalog does not yet include eccDNAs from CIRCLE-seq or CIDER-Seq since few sequencing datasets for those techniques are available

## Evidence

- [abstract] eccDNAdb currently includes 1270 eccDNAs identified in 480 samples of 42 cancers: "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [abstract] A total of 54,901 eccDNA genes were annotated and included in the database: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] Systematic identification of eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [discussion] Manual curation of eccDNAs from published articles to increase database coverage: "Meanwhile, manual curation of eccDNAs from published articles was also performed to increase the database coverage."
- [discussion] Limitation: all eccDNAs in the database were identified by the AmpliconArchitect pipeline only: "will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
- [readme] Repository README: number notation indicates analysis order: "The number notation at the beginning of a script filename indicates the order of an analysis. It is easy to follow the steps to repeat the analyses in our study."
