---
name: peak-qc-metrics-computation
description: Use when ChIP-seq/CUT&Run/CUT&Tag peak files (e.g. BED narrowPeaks) need
  QC before cross-sample comparison, specifically quantifying the fraction of peaks
  falling in ENCODE blacklisted regions.
when_to_use_negative:
- Input is already a blacklist-free/feature count matrix (e.g. peak x sample matrix)
  rather than raw genomic interval peak files
- The goal is peak overlap statistics or functional annotation (ChromHMM/ChIPseeker/TSS
  enrichment) rather than blacklist-region QC filtering and metric computation
- Peaks are not genomic intervals from epigenomic experiments (e.g. ATAC fragment
  files or Picard duplication summaries, which are separate EpiCompare inputs)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EpiCompare
  role: 'Runs the blacklist-filtering QC workflow: tidy_peakfile() removes intervals
    overlapping the ENCODE blacklist; peak_info() computes the blacklist-overlap percentage
    reported in the report''s General Metrics section'
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Language environment in which GRanges peak objects are loaded, tidied, and
    summarized
- name: Bioconductor
  role: Distribution channel for EpiCompare (>= v3.15) and the GRanges/rtracklayer
    infrastructure used for interval overlap
- name: rmarkdown
  role: Renders the QC results, including blacklist percentages, into the single interactive
    HTML report
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:662894ee71c8241e20a5456575150228cd94140904f5234b0312d71568bad405
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# peak-qc-metrics-computation

## Summary

Compute peak-level quality-control metrics for epigenomic peak files by removing blacklisted regions (and non-standard chromosomes) with EpiCompare's tidy_peakfile()/peak_info() workflow and reporting the percentage of peaks overlapping ENCODE blacklist regions in the General Metrics section of the HTML report.

## When to use

Use when ChIP-seq/CUT&Run/CUT&Tag peak files (e.g. BED narrowPeaks) need QC before cross-sample comparison, specifically quantifying the fraction of peaks falling in ENCODE blacklisted regions.

## When NOT to use

- Input is already a blacklist-free/feature count matrix (e.g. peak x sample matrix) rather than raw genomic interval peak files
- The goal is peak overlap statistics or functional annotation (ChromHMM/ChIPseeker/TSS enrichment) rather than blacklist-region QC filtering and metric computation
- Peaks are not genomic intervals from epigenomic experiments (e.g. ATAC fragment files or Picard duplication summaries, which are separate EpiCompare inputs)

## Inputs

- Named list of peak files as GRanges objects or paths to BED-format peak files (e.g. *_peaks.stringent.bed)
- Genome build specification (hg19 or hg38) per peak file via genome_build
- ENCODE blacklist peak file (built-in hg19/hg38 datasets from ENCODE, e.g. accession ENCFF044JNJ)

## Outputs

- Tidied peak intervals in BED format (tidied_peaks.bed)
- QC metrics summary table (peak_qc_summary.tsv) including percentage of peaks overlapping blacklisted regions
- General Metrics entries in the EpiCompare HTML report: percentage of blacklisted and non-standard peaks, and peak widths

## How to apply

Load each input peak file into R as a GRanges object (EpiCompare::gather_files or rtracklayer/GenomicRanges accept BED paths). Select the genome-appropriate ENCODE blacklist supplied by EpiCompare — hg19 or hg38 — and supply it with the genome_build parameter, matching genome_build_output to the target build. Apply tidy_peakfile() to remove peak intervals overlapping blacklisted regions and non-standard chromosomes, then run peak_info() on the tidied file to compute QC metrics including the percentage of peaks in blacklisted regions. This filtering is always performed prior to analysis so downstream overlap, correlation, and annotation plots are computed on blacklist-free peaks. Write the tidied intervals to tidied_peaks.bed and the QC metrics to peak_qc_summary.tsv; the blacklist-overlap percentage is reported per peak file in the General Metrics section of the single interactive HTML report.

## Related tools

- **EpiCompare** (Runs the blacklist-filtering QC workflow: tidy_peakfile() removes intervals overlapping the ENCODE blacklist; peak_info() computes the blacklist-overlap percentage reported in the report's General Metrics section) — https://github.com/neurogenomics/EpiCompare
- **R** (Language environment in which GRanges peak objects are loaded, tidied, and summarized)
- **Bioconductor** (Distribution channel for EpiCompare (>= v3.15) and the GRanges/rtracklayer infrastructure used for interval overlap)
- **rmarkdown** (Renders the QC results, including blacklist percentages, into the single interactive HTML report)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = "hg19", genome_build_output = "hg19", output_dir = tempdir(), run_all = TRUE)
```

## Evaluation signals

- tidied_peaks.bed exists and parses as a valid BED-format interval file
- peak_qc_summary.tsv reports a blacklist-overlap percentage for each peak file with a value between 0 and 100
- No intervals in the tidied peak file overlap the supplied ENCODE blacklist (hg19 or hg38) regions and no non-standard chromosomes remain
- The General Metrics section of the HTML report lists the percentage of blacklisted (and non-standard) peaks for every input peak file
- Peak counts in the tidied file are <= the original file's peak count, with the difference accounted for by blacklist/non-standard-chromosome removal

## Limitations

- Blacklist removal requires the user to supply a blacklist file and a genome build; a mismatch between peakfile genome build (e.g. hg19 peaks with the hg38 blacklist) will yield wrong overlap percentages
- Peaks in non-standard chromosomes are also removed, so QC metrics exclude those intervals by design and cannot be recovered without re-running with different inputs
- The blacklist-overlap percentage is a pre-analysis QC statistic only; it does not by itself benchmark peak quality against a reference dataset

## Evidence

- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: [evidence span withheld — anchor status: external_doc]
- [readme] General Metrics section content: "General Metrics: Metrics on peaks (percentage of blacklisted and non-standard peaks, and peak widths) and fragments (duplication rate) of samples."
- [intro] It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE.: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
