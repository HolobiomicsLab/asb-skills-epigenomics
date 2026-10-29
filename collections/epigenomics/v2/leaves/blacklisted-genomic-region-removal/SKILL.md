---
name: blacklisted-genomic-region-removal
description: Use when input peak files (BED/GRanges, e.g. ChIP-seq, CUT&Run, CUT&Tag
  peaks) must be cleaned of anomalous-signal regions before comparison/QC, and when
  a blacklist-overlap percentage metric is needed per sample.
when_to_use_negative:
- Input is already a blacklist-filtered/standard-chromosome-only peak file — re-running
  would double-report the blacklist percentage against an already-tidied file
- Analysis goal is not peak QC/benchmarking but, e.g., sequence-level variant filtering
  — the blacklist removal here applies only to epigenomic peak intervals
- Genome build is not hg19 or hg38 and no user-supplied blacklist equivalent is available
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3947
tools:
- name: EpiCompare
  role: Provides the ENCODE hg19/hg38 blacklist datasets, tidies peakfiles by removing
    blacklisted/non-standard peaks, and reports the blacklist percentage in the General
    Metrics section
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment in which peak files are loaded as GRanges objects and
    filtered
- name: Bioconductor
  role: Distribution channel for EpiCompare (>= v3.15) and its GRanges/rtracklayer
    dependencies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:4fbb7deaba456688577328d47ece96cf0917e70ee7b8347674c605eb5f2b7f18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - Bioconductor
  - EpiCompare
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3947
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# blacklisted-genomic-region-removal

## Summary

Remove peak intervals overlapping ENCODE blacklisted genomic regions (hg19 or hg38) from epigenomic peak files as EpiCompare's first quality-control step, and report the percentage of peaks falling in blacklist regions in the General Metrics section of the HTML report.

## When to use

Use when input peak files (BED/GRanges, e.g. ChIP-seq, CUT&Run, CUT&Tag peaks) must be cleaned of anomalous-signal regions before comparison/QC, and when a blacklist-overlap percentage metric is needed per sample.

## When NOT to use

- Input is already a blacklist-filtered/standard-chromosome-only peak file — re-running would double-report the blacklist percentage against an already-tidied file
- Analysis goal is not peak QC/benchmarking but, e.g., sequence-level variant filtering — the blacklist removal here applies only to epigenomic peak intervals
- Genome build is not hg19 or hg38 and no user-supplied blacklist equivalent is available

## Inputs

- Peak files (BED format or GRanges objects) from epigenomic assays such as ChIP-seq, CUT&Run, CUT&Tag
- Genome build identifier for each peakfile (hg19 or hg38), e.g. genome_build = list(peakfiles="hg19", reference="hg38")
- Built-in ENCODE blacklist dataset for hg19 or hg38 (supplied by EpiCompare)

## Outputs

- Tidied peak files with blacklisted and non-standard-chromosome peaks removed
- Per-sample QC metric: percentage of peaks overlapping blacklisted regions (0–100), reported in the General Metrics section
- Single interactive HTML EpiCompare report

## How to apply

Import each input peak file into R as a GRanges object (e.g. via rtracklayer/GenomicRanges, or EpiCompare::gather_files for directories of BED files). Select the built-in ENCODE blacklist matching the genome build — EpiCompare ships hg19 and hg38 blacklisted regions obtained from ENCODE. Apply EpiCompare's tidying (tidy_peakfile) so that peak intervals overlapping blacklisted regions — and non-standard chromosomes — are removed prior to analysis. Pass the tidied peakfiles into EpiCompare::EpiCompare() along with the matching genome_build; the function computes peak QC metrics internally, and the percentage of peaks in blacklisted and non-standard regions is surfaced in the report's 'General Metrics' section. Because blacklist filtering is the first QC step, do not compare or annotate peaks before this removal is applied.

## Related tools

- **EpiCompare** (Provides the ENCODE hg19/hg38 blacklist datasets, tidies peakfiles by removing blacklisted/non-standard peaks, and reports the blacklist percentage in the General Metrics section) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment in which peak files are loaded as GRanges objects and filtered)
- **Bioconductor** (Distribution channel for EpiCompare (>= v3.15) and its GRanges/rtracklayer dependencies)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = list(peakfiles="hg19", reference="hg38"), genome_build_output = "hg19", output_dir = tempdir(), run_all = TRUE)
```

## Evaluation signals

- Tidied peak output exists and parses as a BED-format interval file
- QC summary reports a blacklist-overlap percentage strictly between 0 and 100 per peak file
- No intervals in the tidied peak file overlap the applicable ENCODE hg19/hg38 blacklist dataset
- Peak counts in the tidied file are <= original counts, with the difference attributable to blacklisted regions and non-standard chromosomes
- The blacklist percentage appears in the 'General Metrics' section of the single HTML report

## Limitations

- Only hg19 and hg38 ENCODE blacklists are bundled; other genome builds require user-supplied equivalents
- Blacklist filtering is a built-in, unconditional preprocessing step — peaks in blacklisted regions are removed prior to analysis whether or not the user wants them retained
- Removal is applied jointly with non-standard-chromosome filtering, so QC percentages of the two causes are aggregated in the metric

## Evidence

- [intro] intro: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
