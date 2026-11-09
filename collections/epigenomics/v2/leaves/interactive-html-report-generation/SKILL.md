---
name: interactive-html-report-generation
description: Use when comparing multiple epigenomic peak files (e.g. ATAC-seq vs DNase-seq,
  or CUT&Tag/CUT&Run vs ENCODE ChIP-seq H3K27ac reference) and you need QC, overlap
  statistics, and functional annotation collated in one report.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) — EpiCompare operates on peak files (narrowPeak/BED)
  and Picard summaries, not raw reads or alignment
- You need single-sample QC only with no comparison reference or second dataset —
  the skill's core value is contrasting datasets, and overlap/precision-recall sections
  require multiple peak files
- Peak files have not been blacklist-filtered and you plan downstream analyses on
  the peaks themselves — EpiCompare removes blacklisted and non-standard-chromosome
  peaks internally, so its outputs should not be used as a substitute for returning
  cleaned peak files
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0085
tools:
- name: EpiCompare
  role: Core Bioconductor R package that runs the one-line comparison/QC workflow
    and renders the collated HTML report
  repo: https://github.com/neurogenomics/EpiCompare
- name: ChIPseeker
  role: Provides the functional annotation of peaks used in the report's Functional
    Annotation section
- name: knitr
  role: Vignette/report rendering engine for the HTML output
- name: rmarkdown
  role: Vignette/report rendering engine for the HTML output
- name: BiocStyle
  role: HTML document style (BiocStyle::html_document) used when rendering the report
- name: EpiArchives
  role: Public GitHub Pages archive of reference EpiCompare HTML reports (e.g. atac_dnase_chip_example)
    plus reproduction code, used to validate generated reports
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:49179ee5921f3a16622aca68a8dca5152aa0c4d616fb3bb554dc016887dd3e32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - Bioconductor
  - ChIPseeker
  - EpiArchives
  - EpiCompare
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# interactive-html-report-generation

## Summary

Generate a single collated interactive HTML report of epigenomic peak-file QC and benchmarking using the Bioconductor R package EpiCompare, which wraps peak overlap, ChromHMM/ChIPseeker annotation, and Picard-derived metrics into one knitr/rmarkdown (BiocStyle::html_document) document.

## When to use

Use when comparing multiple epigenomic peak files (e.g. ATAC-seq vs DNase-seq, or CUT&Tag/CUT&Run vs ENCODE ChIP-seq H3K27ac reference) and you need QC, overlap statistics, and functional annotation collated in one report.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) — EpiCompare operates on peak files (narrowPeak/BED) and Picard summaries, not raw reads or alignment
- You need single-sample QC only with no comparison reference or second dataset — the skill's core value is contrasting datasets, and overlap/precision-recall sections require multiple peak files
- Peak files have not been blacklist-filtered and you plan downstream analyses on the peaks themselves — EpiCompare removes blacklisted and non-standard-chromosome peaks internally, so its outputs should not be used as a substitute for returning cleaned peak files

## Inputs

- narrowPeak/BED peak files as GRanges objects or file paths (e.g. ENCODE ATAC-seq ENCFF558BLC, ENCFF333TAT; DNase-seq ENCFF274YGF, ENCFF185XRG)
- reference peak file (e.g. ENCODE ChIP-seq H3K27ac ENCFF038DDS or built-in encode_H3K27ac)
- ENCODE blacklist region files for hg19 and hg38
- Picard alignment summary .txt files (e.g. built-in CnT_H3K27ac_picard, CnR_H3K27ac_picard)
- genome build identifiers (hg19/hg38) for peakfiles, reference, and output

## Outputs

- Single interactive collated HTML report (e.g. EpiCompare_report.html) rendered with BiocStyle::html_document
- General Metrics section: peak counts, percentage of blacklisted and non-standard peaks, peak width distributions, fragment duplication rates
- Peak Overlap section: upset plots, precision-recall plots, correlation plots, and overlap significance statistics
- Functional Annotation section: ChromHMM annotations, ChIPseeker annotation, enrichment analysis, and peak enrichment around TSS
- Reproducible R code (e.g. EpiCompare_code.R archived in EpiArchives)

## How to apply

Import narrowPeak/BED peak files into GRanges objects (EpiCompare::gather_files can import BED and Picard summary files) and remove peaks overlapping ENCODE hg19/hg38 blacklist regions and non-standard chromosomes prior to analysis. Assemble a named list of query peakfiles, an optional named reference (e.g. ENCODE H3K27ac ENCFF038DDS), and optional Picard alignment summary files, then call EpiCompare::EpiCompare() once with genome_build (a per-input list like list(peakfiles='hg19', reference='hg38') when builds differ) and genome_build_output, rendering with knitr::rmarkdown under BiocStyle::html_document. The package is designed to run 'with just one line of code' and requires minimal user input. Judge success by checking the output HTML contains the three sections — General Metrics (blacklisted/non-standard peak percentage, peak widths, duplication rate), Peak Overlap (Upset, precision-recall, correlation plots with frequency/percentage/significance), and Functional Annotation (ChromHMM, ChIPseeker, TSS enrichment) — and, where available, by comparing metrics against the published reference report in neurogenomics/EpiArchives.

## Related tools

- **EpiCompare** (Core Bioconductor R package that runs the one-line comparison/QC workflow and renders the collated HTML report) — https://github.com/neurogenomics/EpiCompare
- **ChIPseeker** (Provides the functional annotation of peaks used in the report's Functional Annotation section)
- **knitr** (Vignette/report rendering engine for the HTML output)
- **rmarkdown** (Vignette/report rendering engine for the HTML output)
- **BiocStyle** (HTML document style (BiocStyle::html_document) used when rendering the report)
- **EpiArchives** (Public GitHub Pages archive of reference EpiCompare HTML reports (e.g. atac_dnase_chip_example) plus reproduction code, used to validate generated reports)

## Examples

```
EpiCompare::EpiCompare(peakfiles=list(CnT=CnT_H3K27ac,CnR=CnR_H3K27ac), reference=list(ENCODE_H3K27ac=encode_H3K27ac), genome_build=list(peakfiles="hg19",reference="hg38"), genome_build_output="hg19", picard_files=picard_files, output_dir=tempdir())
```

## Evaluation signals

- The rendered EpiCompare_report.html exists and contains all three sections: General Metrics, Peak Overlap, and Functional Annotation
- General Metrics report peak counts and width distributions consistent with the input narrowPeak files after removal of hg38-blacklisted and non-standard-chromosome peaks
- Peak Overlap section includes upset, precision-recall, and correlation plots with frequency, percentage, and statistical significance of overlapping peaks for every query-vs-reference pair
- For the K562 ATAC-seq vs DNase-seq example, ChromHMM annotations and the precision-recall plot visibly differentiate the two assays, matching the published EpiArchives report 'atac_dnase_chip_example'
- Report renders under BiocStyle::html_document via knitr::rmarkdown without errors and collates results for all supplied query peak files in a single file

## Limitations

- Peaks in ENCODE blacklist regions and on non-standard chromosomes are removed prior to analysis, so report statistics reflect filtered peak sets
- Genome build mismatches must be handled explicitly via the genome_build list and genome_build_output; mixing builds without liftover will invalidate comparisons
- Picard-derived metrics (e.g. duplication rate) require supplying Picard alignment summary files; without them the fragment-level General Metrics are unavailable
- Older Bioconductor versions of EpiCompare may have different functions and parameters than the development-version documentation

## Evidence

- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [abstract] single interactive HTML report: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] three sections README: "General Metrics**: Metrics on peaks (percentage of blacklisted and non-standard peaks, and peak widths) and fragments (duplication rate) of samples."
- [readme] blacklist removal README: [evidence span withheld — anchor status: external_doc]
- [readme] README invocation: "EpiCompare::EpiCompare(peakfiles = peakfiles,
                       genome_build = list(peakfiles="hg19",
                                           reference="hg38"),"
- [other] EpiArchives reference reports: "public archive for interactive HTML reports generated by [`EpiCompare`](https://github.com/neurogenomics/EpiCompare) and the associated code used to create them."
