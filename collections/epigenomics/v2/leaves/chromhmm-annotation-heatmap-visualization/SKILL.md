---
name: chromhmm-annotation-heatmap-visualization
description: Use when comparing open-chromatin or ChIP peak sets (e.g. ATAC-seq vs
  DNase-seq narrowPeak files) and you need a ChromHMM-based functional annotation
  heatmap showing the percentage of peaks overlapping each chromatin state per sample.
when_to_use_negative:
- The goal is peak calling or chromatin-state segmentation itself — EpiCompare consumes
  already-called peak files and pre-existing chromHMM annotations, it does not generate
  them.
- Input data is raw reads (FASTQ) or fragments rather than peak files; the QC/comparison
  functions operate on narrowPeak/BED peaks and Picard summary outputs.
- A single-sample analysis with no comparison target — the ChromHMM heatmap's value
  is contrasting per-state overlap percentages across peak files.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: EpiCompare
  role: Bioconductor R package whose chromHMM plotting function intersects each GRanges
    peak file with chromHMM cell-line annotations, tabulates per-state overlap percentages,
    and renders the annotation heatmap in the HTML report
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment for loading narrowPeak files, standardizing them as GRanges
    objects, and executing the EpiCompare report workflow
- name: knitr
  role: Vignette/report engine used with rmarkdown to build the single collated HTML
    report containing the ChromHMM heatmap
- name: rmarkdown
  role: Report rendering backend (VignetteEngine knitr::rmarkdown) for the EpiCompare
    HTML output
- name: BiocStyle
  role: Document styling (BiocStyle::html_document) for the EpiCompare vignette/report
    output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:22b93c9e65199a6b528b230517e84f42253b962dd14ae7680bafd72add01ae2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - EpiCompare
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chromhmm-annotation-heatmap-visualization

## Summary

Visualize and compare the chromatin-state distribution of epigenomic peak sets by intersecting each peak file with ChromHMM cell-line annotations and rendering the per-state overlap percentages as a heatmap, as implemented in EpiCompare's Functional Annotation report section.

## When to use

Use when comparing open-chromatin or ChIP peak sets (e.g. ATAC-seq vs DNase-seq narrowPeak files) and you need a ChromHMM-based functional annotation heatmap showing the percentage of peaks overlapping each chromatin state per sample.

## When NOT to use

- The goal is peak calling or chromatin-state segmentation itself — EpiCompare consumes already-called peak files and pre-existing chromHMM annotations, it does not generate them.
- Input data is raw reads (FASTQ) or fragments rather than peak files; the QC/comparison functions operate on narrowPeak/BED peaks and Picard summary outputs.
- A single-sample analysis with no comparison target — the ChromHMM heatmap's value is contrasting per-state overlap percentages across peak files.

## Inputs

- Peak files as GRanges objects or paths to narrowPeak/BED files (e.g. K562 ATAC-seq and DNase-seq narrowPeak, ENCODE accession ENCFF044JNJ)
- Cell-line-specific chromHMM annotation file (e.g. K562 chromHMM segments)
- Named list of peakfiles with associated genome_build (hg19/hg38)
- Genome build designation and output directory for the HTML report

## Outputs

- ChromHMM annotation heatmap of chromatin-state overlap percentages per peak file
- Tabulation of percentage of peaks overlapping each chromatin state
- Single interactive HTML report (Functional Annotation section) collating all plots and tables

## How to apply

Standardize each input peak file (e.g. K562 ATAC-seq and DNase-seq ENCODE narrowPeak files such as ENCFF044JNJ) as GRanges objects with EpiCompare; peaks in ENCODE blacklist regions (hg19/hg38) and non-standard chromosomes are removed prior to analysis. Supply the cell-line-specific chromHMM annotation so each peak set is intersected with the chromatin-state segments and the percentage of peaks overlapping each state is tabulated. Render the ChromHMM heatmap with EpiCompare's chromHMM plotting function, then contrast per-state overlap percentages between peak files (e.g. to reproduce the differential annotation seen in Fig. 1d for ATAC-seq vs DNase-seq). This typically runs as part of the single EpiCompare() call with run_all=TRUE, which produces one interactive HTML report collating all results.

## Related tools

- **EpiCompare** (Bioconductor R package whose chromHMM plotting function intersects each GRanges peak file with chromHMM cell-line annotations, tabulates per-state overlap percentages, and renders the annotation heatmap in the HTML report) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment for loading narrowPeak files, standardizing them as GRanges objects, and executing the EpiCompare report workflow)
- **knitr** (Vignette/report engine used with rmarkdown to build the single collated HTML report containing the ChromHMM heatmap)
- **rmarkdown** (Report rendering backend (VignetteEngine knitr::rmarkdown) for the EpiCompare HTML output)
- **BiocStyle** (Document styling (BiocStyle::html_document) for the EpiCompare vignette/report output)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = list(peakfiles="hg19", reference="hg38"), genome_build_output = "hg19", run_all = TRUE, output_dir = tempdir())
```

## Evaluation signals

- Each peak file is confirmed to be a GRanges object with a declared genome build (hg19/hg38), and blacklisted-region and non-standard-chromosome peaks are removed before intersection.
- The report contains a ChromHMM heatmap whose rows/columns cover both peak files and the chromHMM states of the supplied cell line.
- Per-state overlap percentages can be tabulated and contrasted; reproducing the K562 ATAC-seq vs DNase-seq example yields a differential chromatin-state distribution consistent with Fig. 1d of the paper.
- The output is a single interactive HTML report containing the General Metrics, Peak Overlap, and Functional Annotation sections (ChromHMM, ChIPseeker, TSS enrichment).
- Reference report/code on the EpiArchives site (atac_dnase_example) can be diffed against the reproduced report.

## Limitations

- Documentation in the README and GitHub Pages website pertains to the development version of EpiCompare; older Bioconductor versions may have different functions/parameters.
- Peaks in blacklisted regions and non-standard chromosomes are removed prior to analysis, so counts may differ from raw input files.
- The chromHMM annotation must match the cell line of interest; EpiCompare only bundles hg19/hg38 blacklist regions, not all cell-line chromHMM segmentations.

## Evidence

- [methods] ChromHMM annotation heatmap from Fig. 1d: [evidence span withheld — anchor status: not_found]
- [readme] Functional Annotation report section: "**Functional Annotation**: Functional annotation (ChromHMM, ChIPseeker and enrichment analysis) of peaks. Also includes peak enrichment around Transcription Start Site."
- [readme] blacklist filtering rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] single HTML report output: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] reference archive for validation: "`EpiArchives` is a public archive for interactive HTML reports generated by [`EpiCompare`](https://github.com/neurogenomics/EpiCompare) and the associated code used to create them."
