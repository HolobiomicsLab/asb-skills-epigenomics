---
name: precision-recall-curve-computation
description: Use when two or more open-chromatin (or ChIP/CUT&Tag) peak files must
  be benchmarked against a gold-standard reference peak set, and a precision-recall
  plot is needed to contrast technologies such as ATAC-seq vs DNase-seq.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/fragments) rather than called peak files — EpiCompare
  operates on peak intervals, not reads
- The goal is read-level QC such as duplication rate from Picard summaries without
  peak-overlap benchmarking
- Only one peak file and no reference set exist, so precision and recall against a
  gold-standard cannot be defined
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: Provides peak-file handling, blacklist filtering, peak-overlap benchmarking,
    and the precision-recall plot generation within a single HTML report
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment for importing peak files as GRanges and running the benchmarking
    workflow
- name: Bioconductor
  role: Distribution channel (>= v3.15) from which EpiCompare is installed via BiocManager
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:f2203b4a9a94297335acd3df4a645a995014b27d8ec3b95b6402c9bc1e33e2a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precision-recall-curve-computation

## Summary

Benchmark epigenomic peak files (e.g. K562 ATAC-seq vs DNase-seq) against a reference peak set (e.g. ENCODE DNase-seq ENCFF185XRG) by sweeping overlap thresholds and computing precision, recall, and F1 at each, yielding precision-recall curves that distinguish profiling technologies.

## When to use

Use when two or more open-chromatin (or ChIP/CUT&Tag) peak files must be benchmarked against a gold-standard reference peak set, and a precision-recall plot is needed to contrast technologies such as ATAC-seq vs DNase-seq.

## When NOT to use

- Input is raw sequencing data (FASTQ/fragments) rather than called peak files — EpiCompare operates on peak intervals, not reads
- The goal is read-level QC such as duplication rate from Picard summaries without peak-overlap benchmarking
- Only one peak file and no reference set exist, so precision and recall against a gold-standard cannot be defined

## Inputs

- Named list of query peak files (BED files or GRanges objects), e.g. K562 ATAC-seq and K562 DNase-seq peaks
- Named list containing one reference/gold-standard peak file, e.g. ENCODE DNase-seq ENCFF185XRG
- Genome build specification for peakfiles and reference (e.g. hg19/hg38), plus genome_build_output

## Outputs

- pr_metrics.csv: threshold-grid table of precision, recall, and F1 per query peak file
- precision_recall_curves.png: PR curves for all query methods overlaid on a single plot
- Precision-recall section within the single interactive HTML EpiCompare report

## How to apply

Load the query peak files (e.g. K562 ATAC-seq and K562 DNase-seq peaks) and the reference peak set as named lists of GRanges genomic interval objects via EpiCompare's peak-file handling; peaks in ENCODE blacklist regions (hg19/hg38) and non-standard chromosomes are removed prior to analysis. Sweep a grid of overlap thresholds and, at each threshold, intersect each query peak file with the reference using EpiCompare's peak-overlap benchmarking routine, recording frequency, percentage, and statistical significance of overlapping and non-overlapping peaks. At each threshold compute precision (fraction of query peaks overlapping the reference), recall (fraction of reference peaks recovered), and F1 (their harmonic mean). Draw precision-recall curves for all query methods on a single plot — in the paper this reproduces the Fig. 1e distinction between ATAC-seq and DNase-seq against a DNase-seq reference — and export the threshold-grid metrics as a CSV. Note the same EpiCompare report section also emits Upset and correlation plots alongside the precision-recall plot.

## Related tools

- **EpiCompare** (Provides peak-file handling, blacklist filtering, peak-overlap benchmarking, and the precision-recall plot generation within a single HTML report) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment for importing peak files as GRanges and running the benchmarking workflow)
- **Bioconductor** (Distribution channel (>= v3.15) from which EpiCompare is installed via BiocManager)

## Examples

```
EpiCompare::EpiCompare(peakfiles = list(ATAC=K562_ATAC_peaks, DNase=K562_DNase_peaks), genome_build = list(peakfiles="hg38", reference="hg38"), genome_build_output = "hg38", reference = list(ENCFF185XRG = dnase_ref), run_all = TRUE, output_dir = "results/")
```

## Evaluation signals

- pr_metrics.csv contains one row per (query file, overlap threshold) with finite precision, recall, and F1 values in [0,1]
- F1 at each threshold equals the harmonic mean of that threshold's precision and recall
- Precision-recall plot renders curves for every query peak file (ATAC-seq and DNase-seq) on a single figure, matching Fig. 1e's method distinction
- Blacklisted-region and non-standard-chromosome peaks are excluded before overlap computation (EpiCompare removes them prior to analysis)
- Recall is bounded by reference set size: fraction of reference peaks recovered must approach 1 as the overlap threshold loosens

## Limitations

- Documentation pertains to the development version of EpiCompare; older Bioconductor versions may have different functions/parameters
- Blacklisted and non-standard peaks are silently removed prior to analysis, so metrics exclude those intervals by design
- Distinguishing ATAC-seq vs DNase-seq depends on the choice of reference peak set (here a DNase-seq reference); results are reference-dependent

## Evidence

- [readme] Peak Overlap: Frequency, percentage, statistical significance of overlapping and non-overlapping peaks. This also includes Upset, precision-recall and correlation plots.: [evidence span withheld — anchor status: not_found]
- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: [evidence span withheld — anchor status: external_doc]
- [methods] facilitate the comparison of epigenomic datasets for quality control and benchmarking purposes: "facilitate the comparison of epigenomic datasets for quality control and benchmarking purposes"
