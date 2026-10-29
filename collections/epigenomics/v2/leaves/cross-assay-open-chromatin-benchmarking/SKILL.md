---
name: cross-assay-open-chromatin-benchmarking
description: Use when comparing peak files from different chromatin accessibility
  assays (e.g. ATAC-seq vs DNase-seq narrowPeak from ENCODE) to quantify assay-specific
  differences and benchmark against a ChIP-seq gold standard.
when_to_use_negative:
- Input data are not peak files (e.g. raw reads, BAM files, or feature tables) — EpiCompare
  operates on peak files such as narrowPeak/BED
- Goal is within-technology comparison of experimental conditions without a reference
  assay to benchmark against a gold standard like ChIP-seq
- Peaks are not on hg19/hg38 human genome builds or no blacklist file is available,
  since blacklist and non-standard chromosome filtering is central to the QC metrics
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_0749
tools:
- name: EpiCompare
  role: Runs the full one-line comparison and QC report over peakfiles, reference,
    and blacklist
  repo: https://github.com/neurogenomics/EpiCompare
- name: ChIPseeker
  role: Functional annotation of peaks within the report's Functional Annotation section
- name: knitr
  role: Vignette/report rendering engine for the HTML output
- name: rmarkdown
  role: rmarkdown vignette engine rendering the HTML report
- name: BiocStyle
  role: BiocStyle::html_document styling of the rendered report
- name: EpiArchives
  role: Hosts the reference 'atac_dnase_chip_example' report and its reproduction
    code for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:46f4c8f0790062c95c72e9bf5d2f4f7aa11a60dcc9267ade25587480323e3f46
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cross-assay-open-chromatin-benchmarking

## Summary

Benchmark ATAC-seq versus DNase-seq open-chromatin peak profiles from ENCODE K562 cells using EpiCompare's single-call QC report, with a ChIP-seq H3K27ac peak file as reference and ENCODE hg38 blacklist filtering.

## When to use

Use when comparing peak files from different chromatin accessibility assays (e.g. ATAC-seq vs DNase-seq narrowPeak from ENCODE) to quantify assay-specific differences and benchmark against a ChIP-seq gold standard.

## When NOT to use

- Input data are not peak files (e.g. raw reads, BAM files, or feature tables) — EpiCompare operates on peak files such as narrowPeak/BED
- Goal is within-technology comparison of experimental conditions without a reference assay to benchmark against a gold standard like ChIP-seq
- Peaks are not on hg19/hg38 human genome builds or no blacklist file is available, since blacklist and non-standard chromosome filtering is central to the QC metrics

## Inputs

- ENCODE narrowPeak files: ATAC-seq ENCFF558BLC, ENCFF333TAT; DNase-seq ENCFF274YGF, ENCFF185XRG (hg38)
- ENCODE ChIP-seq H3K27ac reference peak file ENCFF038DDS
- ENCODE hg38 blacklist regions file

## Outputs

- Single interactive HTML EpiCompare report with General Metrics, Peak Overlap, and Functional Annotation sections
- Precision-recall and upset plots quantifying ATAC-seq vs DNase-seq peak overlap
- ChromHMM and ChIPseeker functional annotations of each assay's peaks

## How to apply

Download the five ENCODE narrowPeak files (ATAC-seq ENCFF558BLC and ENCFF333TAT, DNase-seq ENCFF274YGF and ENCFF185XRG, ChIP-seq H3K27ac ENCFF038DDS) plus the ENCODE hg38 blacklist. Import peaks into GRanges objects and remove peaks overlapping hg38 blacklisted regions and non-standard chromosomes, since EpiCompare strips these prior to analysis. Run EpiCompare::EpiCompare() in a single call with peakfiles = the ATAC-seq and DNase-seq peaks, genome_build = hg38, blacklist = hg38_blacklist, and reference = the H3K27ac file, rendering under knitr/rmarkdown with BiocStyle::html_document. Inspect the three report sections — General Metrics (peak counts/widths/distributions, blacklist percentage), Peak Overlap (upset, precision-recall, correlation plots), and Functional Annotation (ChromHMM, ChIPseeker, TSS enrichment) — and compare against the published EpiArchives 'atac_dnase_chip_example' report. The largest ATAC-vs-DNase differences appear in the ChromHMM annotations and the precision-recall plot.

## Related tools

- **EpiCompare** (Runs the full one-line comparison and QC report over peakfiles, reference, and blacklist) — https://github.com/neurogenomics/EpiCompare
- **ChIPseeker** (Functional annotation of peaks within the report's Functional Annotation section)
- **knitr** (Vignette/report rendering engine for the HTML output)
- **rmarkdown** (rmarkdown vignette engine rendering the HTML report)
- **BiocStyle** (BiocStyle::html_document styling of the rendered report)
- **EpiArchives** (Hosts the reference 'atac_dnase_chip_example' report and its reproduction code for comparison)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = "hg38", blacklist = hg38_blacklist, reference = list("ENCODE_H3K27ac" = encode_H3K27ac), output_dir = tempdir())
```

## Evaluation signals

- Generated HTML report contains exactly the three sections: General Metrics, Peak Overlap, and Functional Annotation
- Reported peak counts drop after blacklist filtering, and the percentage of blacklisted/non-standard peaks is reported per sample
- Precision-recall and upset plots show measurable ATAC-seq vs DNase-seq divergence, strongest in ChromHMM annotations and the precision-recall plot
- H3K27ac ChIP-seq (ENCFF038DDS) serves as reference against which both accessibility assays are benchmarked
- Metrics visually and quantitatively match the published EpiArchives reference report 'atac_dnase_chip_example'

## Limitations

- Documentation pertains to the development version (1.15.1); older EpiCompare versions may have different functions/parameters, so Bioconductor-version docs should be checked
- Peaks in blacklisted regions and on non-standard chromosomes are removed before analysis, which changes peak counts relative to the raw files
- EpiCompare is designed for epigenomic peak-file comparison, not raw-read processing; Picard summary files are optional inputs for duplication-rate metrics

## Evidence

- [other] Applying EpiCompare to the K562 ATAC-seq and DNase-seq ENCODE datasets reveals differences between the two methods, most evident in ChromHMM annotations and the precision-recall plot.: [evidence span withheld — anchor status: not_found]
- [abstract] The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report.: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [readme] General Metrics: Metrics on peaks (percentage of blacklisted and non-standard peaks, and peak widths) and fragments (duplication rate) of samples.: "**General Metrics**: Metrics on peaks (percentage of blacklisted and
    non-standard peaks, and peak widths) and fragments (duplication
    rate) of samples."
- [readme] This also includes Upset, precision-recall and correlation plots.: "This also includes Upset,
    precision-recall and correlation plots."
- [intro] It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE.: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
