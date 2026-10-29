---
name: cutandrun-cutandtag-peak-quality-control
description: Use when you have CUT&Run, CUT&Tag, or other novel epigenomic peak files
  (e.g. BED/GRanges H3K27ac or ATAC-seq peaks) and need to QC or benchmark them against
  a gold-standard such as ChIP-seq reference peaks.
when_to_use_negative:
- Your goal is de novo peak calling from raw sequencing reads (FASTQ/BAM) — EpiCompare
  operates on already-called peak files, not raw data
- You only have a single peak file and no reference or replicate group — overlap/benchmarking
  comparisons and consensus calling need multiple inputs
- You need a quantitatively validated consensus caller — the paper benchmarks granges
  vs consensusseeker only descriptively, with no numeric criterion reported
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: 'Primary package: filters blacklist/non-standard peaks, computes overlap and
    QC metrics, calls consensus peaks (compute_consensus_peaks), and renders the HTML
    report'
  repo: https://github.com/neurogenomics/EpiCompare
- name: Bioconductor
  role: Distribution platform (>= v3.15) from which EpiCompare is installed via BiocManager
- name: ChIPseeker
  role: Downstream R tool leveraged within EpiCompare for functional annotation of
    peaks in the report
- name: EpiArchives
  role: Public archive of pre-computed EpiCompare HTML reports (e.g. atac_dnase_chip_example)
    with reproduction code for benchmarking reference points
  repo: https://github.com/neurogenomics/EpiArchives
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:0c30c5e5e90c8234bd78340176f43dab6885ad222b3a642ccb29256e372a8588
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - ChIPseeker
  - EpiArchives
  - EpiCompare
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# CUT&Run/CUT&Tag peak quality control

## Summary

Quality control and benchmarking of CUT&Run/CUT&Tag (CnT/CnR) H3K27ac peak files against a gold-standard ChIP-seq reference using the EpiCompare R package, which collates blacklist/non-standard peak percentages, peak widths, duplication rates, and peak-overlap statistics into a single interactive HTML report.

## When to use

Use when you have CUT&Run, CUT&Tag, or other novel epigenomic peak files (e.g. BED/GRanges H3K27ac or ATAC-seq peaks) and need to QC or benchmark them against a gold-standard such as ChIP-seq reference peaks.

## When NOT to use

- Your goal is de novo peak calling from raw sequencing reads (FASTQ/BAM) — EpiCompare operates on already-called peak files, not raw data
- You only have a single peak file and no reference or replicate group — overlap/benchmarking comparisons and consensus calling need multiple inputs
- You need a quantitatively validated consensus caller — the paper benchmarks granges vs consensusseeker only descriptively, with no numeric criterion reported

## Inputs

- Named list of peak files as GRanges objects or paths to BED files (e.g. *_peaks.stringent.bed)
- Named list of reference peak files (e.g. ENCODE H3K27ac, ENCODE accession ENCFF044JNJ)
- Named list of Picard summary output text files (duplication metrics)
- Genome build specification per input (e.g. hg19/hg38) and target output build
- Output directory for the HTML report

## Outputs

- Single interactive HTML QC report (general metrics, peak overlap, functional annotation sections)
- Cleaned peak sets with ENCODE blacklist and non-standard chromosome peaks removed
- Consensus peak GRanges objects (method='granges' or method='consensusseeker'), exportable as BED files
- UpSet plots, precision-recall plots, correlation plots, ChromHMM/ChIPseeker annotations, TSS enrichment plots

## How to apply

Load the built-in example datasets (CnT_H3K27ac, CnR_H3K27ac, encode_H3K27ac, plus Picard summary outputs CnT_H3K27ac_picard / CnR_H3K27ac_picard) or import user peak files with EpiCompare::gather_files (type='peaks.stringent' for BED, 'picard' for Picard txt), assembling named lists for peakfiles, reference, and picard_files. EpiCompare automatically removes peaks in ENCODE hg19/hg38 blacklist regions and non-standard chromosomes prior to analysis, so QC metrics like percentage of blacklisted and non-standard peaks are computed after this filter. Specify genome_build per input (e.g. list(peakfiles='hg19', reference='hg38')) and genome_build_output to harmonize builds. Optionally derive a consensus peak set from a group of replicates with compute_consensus_peaks using method='granges' (fast interval-overlap) or method='consensusseeker' (slower, models peak score distributions). The function requires minimal input — effectively one line of code — and emits a single interactive HTML report covering general metrics, peak overlap (UpSet, precision-recall, correlation), and functional annotation (ChromHMM, ChIPseeker, TSS enrichment).

## Related tools

- **EpiCompare** (Primary package: filters blacklist/non-standard peaks, computes overlap and QC metrics, calls consensus peaks (compute_consensus_peaks), and renders the HTML report) — https://github.com/neurogenomics/EpiCompare
- **Bioconductor** (Distribution platform (>= v3.15) from which EpiCompare is installed via BiocManager)
- **ChIPseeker** (Downstream R tool leveraged within EpiCompare for functional annotation of peaks in the report)
- **EpiArchives** (Public archive of pre-computed EpiCompare HTML reports (e.g. atac_dnase_chip_example) with reproduction code for benchmarking reference points) — https://github.com/neurogenomics/EpiArchives

## Examples

```
peakfiles <- list(CnT=CnT_H3K27ac, CnR=CnR_H3K27ac); EpiCompare::EpiCompare(peakfiles=peakfiles, genome_build=list(peakfiles='hg19', reference='hg38'), genome_build_output='hg19', run_all=TRUE, output_dir=tempdir())
```

## Evaluation signals

- The single interactive HTML report is produced in output_dir with all three sections (general metrics, peak overlap, functional annotation) populated
- Reported percentages of blacklisted and non-standard peaks are consistent with the ENCODE hg19/hg38 blacklist filter having been applied before analysis
- Consensus peak outputs from both method='granges' and method='consensusseeker' exist, parse as valid GRanges/BED, and contain a non-empty peak set covering the CnT and CnR H3K27ac intervals
- Overlapping vs non-overlapping peak frequencies, percentages, and statistical significance are reported, with UpSet/precision-recall/correlation plots rendered
- Picard duplication rates are present for samples when picard_files are supplied

## Limitations

- The granges vs consensusseeker consensus methods are only benchmarked descriptively in the paper; no numeric acceptance criterion is reported
- Peaks in blacklisted regions and on non-standard chromosomes are removed automatically, so results are computed on filtered sets, not raw inputs
- Documentation in the README and GitHub Pages pertains to the development version (1.15.1); older Bioconductor versions may have different available functions and parameters

## Evidence

- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [intro] To better understand the performance of these novel approaches, it is important that we systematically compare these technologies and benchmark against a "gold-standard".: "To better understand the performance of these novel approaches, it is important that we systematically compare these technologies and benchmark against a "gold-standard"."
- [readme] Metrics on peaks (percentage of blacklisted and non-standard peaks, and peak widths) and fragments (duplication rate) of samples.: "**General Metrics**: Metrics on peaks (percentage of blacklisted and
    non-standard peaks, and peak widths) and fragments (duplication
    rate) of samples."
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [abstract] The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report.: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
