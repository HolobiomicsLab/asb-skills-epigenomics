---
name: atac-dnase-chipseq-peak-overlap-analysis
description: Use when you have ENCODE narrowPeak files from two open-chromatin assays
  (e.g. ATAC-seq vs DNase-seq) in the same cell type and need QC plus overlap/benchmarking
  against a ChIP-seq reference.
when_to_use_negative:
- Input is not genomic peak data (e.g. already-summarized feature tables or count
  matrices) — EpiCompare operates on peak files/GRanges, not pre-aggregated results
- Peaks are on non-standard chromosomes or unfiltered against the ENCODE blacklist
  — the filtering step is expected before interpretation of overlap statistics
- You need a different genome build than the one supplied for the reference — mismatched
  genome_build values must be reconciled (genome_build_output) before overlap statistics
  are meaningful
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0085
tools:
- name: EpiCompare
  role: 'Core package: one-line call runs blacklist filtering, QC metrics, overlap
    plots, and functional annotation into a single HTML report'
  repo: https://github.com/neurogenomics/EpiCompare
- name: ChIPseeker
  role: Underlies the functional annotation section of the report for peak annotation
- name: Bioconductor
  role: Distribution channel; EpiCompare is available for Bioconductor >= v3.15
  repo: https://bioconductor.org/packages/EpiCompare
- name: knitr
  role: Vignette/report rendering engine for the HTML output
- name: rmarkdown
  role: Vignette/report rendering engine for the HTML output
- name: BiocStyle
  role: HTML document style used for report rendering
- name: EpiArchives
  role: Public archive hosting the reference report 'atac_dnase_chip_example' and
    its reproduction code for comparison
  repo: https://github.com/neurogenomics/EpiArchives
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:6edc98b421708797c540f4026ad56b6b32e320f9c03eab65ae9b8feee8994ad6
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# atac-dnase-chipseq-peak-overlap-analysis

## Summary

Run EpiCompare to contrast ATAC-seq and DNase-seq open-chromatin peak sets (with a H3K27ac ChIP-seq reference) in K562 cells, producing a single interactive HTML report with General Metrics, Peak Overlap (upset, precision-recall, correlation), and Functional Annotation (ChromHMM, ChIPseeker) sections.

## When to use

Use when you have ENCODE narrowPeak files from two open-chromatin assays (e.g. ATAC-seq vs DNase-seq) in the same cell type and need QC plus overlap/benchmarking against a ChIP-seq reference.

## When NOT to use

- Input is not genomic peak data (e.g. already-summarized feature tables or count matrices) — EpiCompare operates on peak files/GRanges, not pre-aggregated results
- Peaks are on non-standard chromosomes or unfiltered against the ENCODE blacklist — the filtering step is expected before interpretation of overlap statistics
- You need a different genome build than the one supplied for the reference — mismatched genome_build values must be reconciled (genome_build_output) before overlap statistics are meaningful

## Inputs

- ENCODE narrowPeak peak files (BED/narrowPeak): ATAC-seq ENCFF558BLC, ENCFF333TAT; DNase-seq ENCFF274YGF, ENCFF185XRG
- H3K27ac ChIP-seq narrowPeak reference file ENCFF038DDS
- ENCODE hg38 blacklist regions file
- Optional: Picard summary output txt files for duplication-rate metrics
- Genome build annotation (hg38)

## Outputs

- Single interactive HTML report (knitr/rmarkdown, BiocStyle::html_document) with General Metrics, Peak Overlap, and Functional Annotation sections
- Upset, precision-recall, and correlation overlap plots across ATAC-seq, DNase-seq, and H3K27ac ChIP-seq peaks
- ChromHMM and ChIPseeker functional annotations of peaks, plus TSS enrichment
- Blacklist- and non-standard-chromosome-filtered GRanges peak objects

## How to apply

Install EpiCompare from Bioconductor (>= v3.15) and download the five ENCODE files: ATAC-seq narrowPeaks ENCFF558BLC and ENCFF333TAT, DNase-seq ENCFF274YGF and ENCFF185XRG, H3K27ac ChIP-seq ENCFF038DDS, plus the ENCODE hg38 blacklist. Import the peaks as GRanges objects (EpiCompare::gather_files can import BED/narrowPeak files) and remove peaks overlapping hg38 blacklisted regions and non-standard chromosomes — EpiCompare does this filtering prior to analysis. Call EpiCompare::EpiCompare() in one line with peakfiles = the filtered ATAC-seq/DNase-seq peaks, genome_build = hg38, blacklist = the hg38 blacklist, and reference = the H3K27ac ChIP-seq file, rendering the report via knitr/rmarkdown under BiocStyle::html_document. The generated HTML report has three sections — General Metrics (peak counts, blacklist/non-standard peak percentages, peak widths, duplication rate via Picard input), Peak Overlap (upset, precision-recall, and correlation plots with statistical significance), and Functional Annotation (ChromHMM, ChIPseeker, TSS enrichment) — and can be compared against the published EpiArchives report 'atac_dnase_chip_example'.

## Related tools

- **EpiCompare** (Core package: one-line call runs blacklist filtering, QC metrics, overlap plots, and functional annotation into a single HTML report) — https://github.com/neurogenomics/EpiCompare
- **ChIPseeker** (Underlies the functional annotation section of the report for peak annotation)
- **Bioconductor** (Distribution channel; EpiCompare is available for Bioconductor >= v3.15) — https://bioconductor.org/packages/EpiCompare
- **knitr** (Vignette/report rendering engine for the HTML output)
- **rmarkdown** (Vignette/report rendering engine for the HTML output)
- **BiocStyle** (HTML document style used for report rendering)
- **EpiArchives** (Public archive hosting the reference report 'atac_dnase_chip_example' and its reproduction code for comparison) — https://github.com/neurogenomics/EpiArchives

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = "hg38", blacklist = hg38_blacklist, reference = list("ENCODE_H3K27ac" = encode_H3K27ac), output_dir = tempdir())
```

## Evaluation signals

- The generated HTML report contains all three expected sections: General Metrics, Peak Overlap, and Functional Annotation
- Blacklist filtering was applied: peak counts in the report reflect removal of hg38 blacklisted-region and non-standard-chromosome peaks
- Peak Overlap section includes upset, precision-recall, and correlation plots across the ATAC-seq, DNase-seq, and H3K27ac reference peak sets
- Report metrics (peak counts, widths, ChromHMM annotations, precision-recall) are quantitatively comparable to the published EpiArchives reference report 'atac_dnase_chip_example'
- Differences between ATAC-seq and DNase-seq are visible, most evident in ChromHMM annotations and the precision-recall plot

## Limitations

- EpiCompare compares epigenomic peak datasets (ChIP-seq, CUT&Run, CUT&Tag, ATAC-seq, DNase-seq) — it is not a general-purpose peak caller or aligner
- Peaks in blacklisted regions and on non-standard chromosomes are removed prior to analysis, so counts differ from raw input files
- Documentation in the README/website pertains to the development version (1.15.1); older Bioconductor versions may have different functions and parameters
- Reference and peakfile genome builds may differ and must be specified (e.g. list(peakfiles='hg19', reference='hg38')) with a chosen genome_build_output

## Evidence

- [abstract] EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets.: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [other] Applying EpiCompare to the K562 ATAC-seq and DNase-seq ENCODE datasets reveals differences between the two methods, most evident in ChromHMM annotations and the precision-recall plot.: [evidence span withheld — anchor status: not_found]
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [abstract] One-line execution producing a single HTML report: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] Report structure described in README: "The function outputs a
report in HTML format consisting of three sections:

1.  **General Metrics**: Metrics on peaks (percentage of blacklisted and
    non-standard peaks, and peak widths) and"
