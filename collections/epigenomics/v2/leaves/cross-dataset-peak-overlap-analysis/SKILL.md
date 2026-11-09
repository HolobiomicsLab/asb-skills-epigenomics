---
name: cross-dataset-peak-overlap-analysis
description: Use when multiple peak files from different epigenomic assays (e.g. CUT&Tag,
  CUT&Run, ChIP-seq BED/narrowPeak) must be benchmarked against a reference for QC,
  overlap statistics, and functional annotation.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) rather than already-called peak files —
  EpiCompare starts from peak files, not alignments
- Only a single peak file with no reference or comparison dataset exists — the overlap/benchmarking
  sections need multiple query files or a reference
- Peaks are on a genome build with no ENCODE blacklist available (e.g. non-human hg19/hg38
  genomes)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: EpiCompare
  role: 'Core package: takes named lists of query peakfiles, reference peakfile, blacklist,
    and Picard summaries and generates the collated HTML report'
  repo: https://github.com/neurogenomics/EpiCompare
- name: knitr
  role: Vignette/report rendering engine behind the HTML report
- name: rmarkdown
  role: Vignette engine used to render the EpiCompare report
- name: BiocStyle
  role: HTML document style (BiocStyle::html_document) for the rendered report
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:2838aa7d695c126a03d32d6947cb17e75144c4287810cfd347126815b5c203a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - EpiCompare
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-dataset-peak-overlap-analysis

## Summary

Run EpiCompare to compare epigenomic peak files across datasets and technologies (e.g. CUT&Tag and CUT&Run H3K27ac against an ENCODE ChIP-seq reference), producing a single interactive HTML report with General Metrics, Peak Overlap, and Functional Annotation sections.

## When to use

Use when multiple peak files from different epigenomic assays (e.g. CUT&Tag, CUT&Run, ChIP-seq BED/narrowPeak) must be benchmarked against a reference for QC, overlap statistics, and functional annotation.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) rather than already-called peak files — EpiCompare starts from peak files, not alignments
- Only a single peak file with no reference or comparison dataset exists — the overlap/benchmarking sections need multiple query files or a reference
- Peaks are on a genome build with no ENCODE blacklist available (e.g. non-human hg19/hg38 genomes)

## Inputs

- Query peak files (e.g. CnT_H3K27ac, CnR_H3K27ac) as GRanges objects or BED file paths
- Reference peak file (ENCODE H3K27ac, accession ENCFF044JNJ)
- ENCODE blacklist region file for hg19 or hg38
- Picard alignment summary txt files (e.g. CnT_H3K27ac_picard, CnR_H3K27ac_picard)
- Genome build annotations per input (e.g. list(peakfiles="hg19", reference="hg38"))

## Outputs

- Single collated interactive HTML report (EpiCompare_report.html) with General Metrics, Peak Overlap, and Functional Annotation sections
- Peak overlap statistics: frequency, percentage, and statistical significance of overlapping and non-overlapping peaks

## How to apply

Load EpiCompare's example or user-supplied peak files as named lists of GRanges (CnT_H3K27ac for CUT&Tag, CnR_H3K27ac for CUT&Run, and the ENCODE ENCFF044JNJ H3K27ac reference), plus hg19/hg38 ENCODE blacklist regions and Picard alignment summary files for fragment/duplication metrics. Because blacklisted regions and non-standard chromosomes are removed prior to analysis, and genome builds may differ between query and reference, set genome_build as a list per input and fix genome_build_output (e.g. "hg19") to harmonize coordinates. Call EpiCompare() once with peakfiles, reference, picard_files, and output_dir so it collates all results into one HTML report. Judge success by verifying that EpiCompare_report.html exists and contains all three sections: General Metrics (blacklisted/non-standard peak percentage, peak widths, duplication rate), Peak Overlap (Upset, precision-recall, correlation plots), and Functional Annotation (ChromHMM, ChIPseeker, TSS enrichment).

## Related tools

- **EpiCompare** (Core package: takes named lists of query peakfiles, reference peakfile, blacklist, and Picard summaries and generates the collated HTML report) — https://github.com/neurogenomics/EpiCompare
- **knitr** (Vignette/report rendering engine behind the HTML report)
- **rmarkdown** (Vignette engine used to render the EpiCompare report)
- **BiocStyle** (HTML document style (BiocStyle::html_document) for the rendered report)

## Examples

```
EpiCompare::EpiCompare(peakfiles=list("CnT"=CnT_H3K27ac,"CnR"=CnR_H3K27ac), reference=list("ENCODE_H3K27ac"=encode_H3K27ac), genome_build_output="hg19", output_dir=tempdir())
```

## Evaluation signals

- EpiCompare_report.html exists in the specified output directory after a single EpiCompare() call
- Report contains all three expected sections: General Metrics, Peak Overlap, and Functional Annotation
- Overlap statistics are computed for both query files (CnT and CnR) against the encode_H3K27ac reference
- Blacklisted-region and non-standard-chromosome peaks are absent from the analyzed files (removed prior to analysis)
- QC metrics (e.g. duplication rate) derived from the supplied Picard summary files appear in the report

## Limitations

- Peaks in blacklisted regions and on non-standard chromosomes are removed before analysis, so reported counts reflect filtered peak sets
- Genome builds can differ between query and reference; the user must specify per-input builds and a genome_build_output to avoid spurious non-overlaps
- Documentation in the README/website pertains to the development version; older Bioconductor versions may have different functions and parameters

## Evidence

- [abstract] minimal user input, single interactive HTML report: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [abstract] package purpose: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] Peak Overlap section contents: [evidence span withheld — anchor status: external_doc]
- [readme] pre-analysis filtering: [evidence span withheld — anchor status: external_doc]
- [methods] single collated report: "generates a single report that collates all results of the analysis"
