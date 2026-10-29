---
name: chipseq-functional-annotation-of-peak-features
description: Use when narrowPeak/BED peak files from epigenomic assays (e.g. K562
  ATAC-seq vs DNase-seq vs H3K27ac ChIP-seq) need biological interpretation — genomic
  state (ChromHMM), nearest-gene (ChIPseeker) and TSS-enrichment annotation — rather
  than just QC metrics.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) rather than already-called peak files —
  EpiCompare assumes post-peak-calling narrowPeak/BED inputs
- The goal is only QC metrics (peak counts, widths, blacklist fraction) — the General
  Metrics section, not functional annotation, covers this
- Peak sets are in an unsupported genome build or have not been checked against hg19/hg38
  blacklisted regions, since blacklisted and non-standard-chromosome peaks are removed
  prior to analysis
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: Wraps and orchestrates the functional annotation section (ChromHMM, ChIPseeker,
    enrichment, TSS enrichment) inside the collated HTML report; run with one call
  repo: https://github.com/neurogenomics/EpiCompare
- name: ChIPseeker
  role: R package used internally for ChIPseeker-based functional/nearest-gene annotation
    of peaks
- name: ChromHMM
  role: Chromatin-state annotation of peaks as rendered in the Functional Annotation
    section
- name: knitr/rmarkdown/BiocStyle
  role: Vignette/report rendering engine producing the BiocStyle::html_document HTML
    output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:308dd47bcabe06465cf98f410958cc476c2aa74c086906c53512127818d1d274
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - BiocStyle
  - Bioconductor
  - ChIPseeker
  - ChromHMM
  - EpiCompare
  - R
  - knitr
  - knitr/rmarkdown/BiocStyle
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# chipseq-functional-annotation-of-peak-features

## Summary

Annotate epigenomic peak sets (ChIP-seq, ATAC-seq, DNase-seq narrowPeak files) functionally using EpiCompare's built-in ChromHMM, ChIPseeker and enrichment analyses, including peak enrichment around Transcription Start Sites, as rendered in the third section of a single interactive HTML report.

## When to use

Use when narrowPeak/BED peak files from epigenomic assays (e.g. K562 ATAC-seq vs DNase-seq vs H3K27ac ChIP-seq) need biological interpretation — genomic state (ChromHMM), nearest-gene (ChIPseeker) and TSS-enrichment annotation — rather than just QC metrics.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) rather than already-called peak files — EpiCompare assumes post-peak-calling narrowPeak/BED inputs
- The goal is only QC metrics (peak counts, widths, blacklist fraction) — the General Metrics section, not functional annotation, covers this
- Peak sets are in an unsupported genome build or have not been checked against hg19/hg38 blacklisted regions, since blacklisted and non-standard-chromosome peaks are removed prior to analysis

## Inputs

- Peak files (narrowPeak/BED) as GRanges objects or file paths, e.g. ENCODE ATAC-seq ENCFF558BLC/ENCFF333TAT and DNase-seq ENCFF274YGF/ENCFF185XRG
- Reference peak file (e.g. H3K27ac ChIP-seq ENCFF038DDS) for benchmarking
- Genome build specification (hg19 or hg38)
- ENCODE blacklist regions for hg19/hg38 (e.g. ENCFF044JNJ and hg38 blacklist)

## Outputs

- Single interactive HTML report (knitr/rmarkdown, BiocStyle::html_document) with a Functional Annotation section
- ChromHMM chromatin-state annotations of the peak sets
- ChIPseeker functional/nearest-gene annotations of peaks
- Peak enrichment plot around Transcription Start Sites
- Enrichment analysis results of annotated peaks

## How to apply

Import peak files (e.g. ENCODE narrowPeak files such as ENCFF558BLC, ENCFF333TAT for ATAC-seq and ENCFF274YGF, ENCFF185XRG for DNase-seq) as GRanges objects, and first remove peaks overlapping the ENCODE hg38 blacklist and non-standard chromosomes, since EpiCompare strips these prior to analysis. Supply the filtered peakfiles, genome_build (hg19/hg38, e.g. list(peakfiles='hg19', reference='hg38')), blacklist, and an optional reference peak set (e.g. H3K27ac ChIP-seq ENCFF038DDS) to EpiCompare::EpiCompare() in a single call; the functional annotation runs as the third report section alongside General Metrics and Peak Overlap. Inspect the ChromHMM annotations (which most strongly differentiated ATAC-seq from DNase-seq in the K562 example), the ChIPseeker annotation, and the peak enrichment around TSS. Compare the rendered BiocStyle::html_document output against the EpiArchives reference report 'atac_dnase_chip_example' to confirm the annotation section is populated and interpretable.

## Related tools

- **EpiCompare** (Wraps and orchestrates the functional annotation section (ChromHMM, ChIPseeker, enrichment, TSS enrichment) inside the collated HTML report; run with one call) — https://github.com/neurogenomics/EpiCompare
- **ChIPseeker** (R package used internally for ChIPseeker-based functional/nearest-gene annotation of peaks)
- **ChromHMM** (Chromatin-state annotation of peaks as rendered in the Functional Annotation section)
- **knitr/rmarkdown/BiocStyle** (Vignette/report rendering engine producing the BiocStyle::html_document HTML output)

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = list(peakfiles="hg38", reference="hg38"), blacklist = hg38_blacklist, reference = list("ENCODE_H3K27ac" = encode_H3K27ac), output_dir = tempdir())
```

## Evaluation signals

- The generated HTML report contains all three sections — General Metrics, Peak Overlap, and Functional Annotation (ChIPseeker-based annotation) — not just QC tables
- ChromHMM annotations are present and, for the K562 ATAC-seq vs DNase-seq case, show the expected differences between the two assays
- Peaks overlapping hg38 blacklist regions and non-standard chromosomes are removed before annotation (report should show non-blacklisted peak percentages)
- A peak-enrichment-around-TSS plot is rendered in the Functional Annotation section
- Report metrics and annotations match the published EpiArchives reference report 'atac_dnase_chip_example'

## Limitations

- Peaks in blacklisted regions and on non-standard chromosomes are removed from the files prior to analysis, so annotation counts will not match raw peak file totals
- Documentation and available parameters differ between EpiCompare versions — older Bioconductor versions may have different functions (see Bioconductor versioned docs)
- Genome builds of peakfiles and the reference can differ and must be declared explicitly (e.g. genome_build = list(peakfiles='hg19', reference='hg38')) or annotation will be misassigned

## Evidence

- [readme] Functional annotation (ChromHMM, ChIPseeker and enrichment analysis) of peaks. Also includes peak enrichment around Transcription Start Site.: "Functional Annotation**: Functional annotation (ChromHMM, ChIPseeker and enrichment analysis) of peaks. Also includes peak enrichment around Transcription Start Site."
- [intro] There are many tools in R (e.g. ChIPseeker) that can be used to conduct downstream analysis and comparison of epigenomic datasets.: "There are many tools in R (e.g. *ChIPseeker*) that can be used to conduct downstream analysis and comparison of epigenomic datasets."
- [readme] Peaks located in blacklisted regions and non-standard chromosomes are removed from the files prior to analysis.: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [abstract] The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report.: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
