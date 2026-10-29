---
name: epigenomic-peak-quality-control
description: Use when you need to compare or QC epigenomic peak files (narrowPeak/BED
  GRanges) across technologies such as ATAC-seq vs DNase-seq, or CUT&Tag/CUT&Run vs
  a ChIP-seq gold-standard reference.
when_to_use_negative:
- When the goal is raw read alignment or peak calling rather than comparison/QC of
  already-called peak files
- When only one peak file with no reference or comparator is available, since the
  package's main functionality is to contrast epigenomic datasets
- When peaks cannot be mapped to a supported genome build (hg19/hg38), since blacklist
  filtering and annotation depend on it
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: EpiCompare
  role: Core Bioconductor package that runs the QC/benchmarking analysis and renders
    the HTML report in one call
  repo: https://github.com/neurogenomics/EpiCompare
- name: ChIPseeker
  role: Provides the functional annotation of peaks within the report's Functional
    Annotation section
- name: knitr/rmarkdown/BiocStyle
  role: Vignette/report rendering engine producing the BiocStyle::html_document output
- name: EpiArchives
  role: Public archive of reference EpiCompare HTML reports (e.g. atac_dnase_chip_example)
    used to benchmark generated reports
  repo: https://github.com/neurogenomics/EpiArchives
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:63d3e6e996cacac0a853fa7e73caf878957d8087ab205bc8d0d4a3ee1c1fd00d
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
  - knitr/rmarkdown/BiocStyle
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# epigenomic-peak-quality-control

## Summary

Run EpiCompare to quality-control and benchmark epigenomic peak files (e.g. ATAC-seq, DNase-seq, CUT&Tag, CUT&Run, ChIP-seq) by generating a single interactive HTML report with General Metrics, Peak Overlap, and Functional Annotation sections.

## When to use

Use when you need to compare or QC epigenomic peak files (narrowPeak/BED GRanges) across technologies such as ATAC-seq vs DNase-seq, or CUT&Tag/CUT&Run vs a ChIP-seq gold-standard reference.

## When NOT to use

- When the goal is raw read alignment or peak calling rather than comparison/QC of already-called peak files
- When only one peak file with no reference or comparator is available, since the package's main functionality is to contrast epigenomic datasets
- When peaks cannot be mapped to a supported genome build (hg19/hg38), since blacklist filtering and annotation depend on it

## Inputs

- GRanges objects or paths to BED/narrowPeak peak files (e.g. ENCODE accessions ENCFF558BLC, ENCFF333TAT for ATAC-seq; ENCFF274YGF, ENCFF185XRG for DNase-seq)
- ENCODE blacklist region files (hg19 and hg38)
- Reference peak file (e.g. ENCODE H3K27ac ChIP-seq ENCFF038DDS)
- Picard alignment summary txt files (optional, for duplication rate)
- Genome build specification (hg19 or hg38) per input set

## Outputs

- Single collated interactive HTML report (EpiCompare_report.html) with General Metrics, Peak Overlap, and Functional Annotation sections
- Peak overlap statistics (frequency, percentage, statistical significance) and UpSet/precision-recall/correlation plots
- ChromHMM, ChIPseeker, and enrichment-based functional annotations of peaks

## How to apply

Install EpiCompare from Bioconductor, import peak files (ENCODE narrowPeak or BED) into GRanges objects, and remove peaks overlapping the ENCODE hg38/hg19 blacklist and non-standard chromosomes before analysis. Call EpiCompare::EpiCompare() in a single line with named lists of peakfiles, an optional reference (e.g. ENCODE H3K27ac ChIP-seq ENCFF038DDS), genome_build (e.g. list(peakfiles='hg19', reference='hg38')), and optional Picard alignment summary files for duplication-rate metrics, rendering with knitr/rmarkdown under BiocStyle::html_document. The report collates General Metrics (peak counts, blacklist/non-standard peak percentages, peak widths, duplication rate), Peak Overlap (UpSet, precision-recall, correlation plots with statistical significance), and Functional Annotation (ChromHMM, ChIPseeker, TSS enrichment). Judge correctness by confirming the single HTML report contains all three sections and comparing metrics against reference EpiArchives reports such as 'atac_dnase_chip_example'.

## Related tools

- **EpiCompare** (Core Bioconductor package that runs the QC/benchmarking analysis and renders the HTML report in one call) — https://github.com/neurogenomics/EpiCompare
- **ChIPseeker** (Provides the functional annotation of peaks within the report's Functional Annotation section)
- **knitr/rmarkdown/BiocStyle** (Vignette/report rendering engine producing the BiocStyle::html_document output)
- **EpiArchives** (Public archive of reference EpiCompare HTML reports (e.g. atac_dnase_chip_example) used to benchmark generated reports) — https://github.com/neurogenomics/EpiArchives

## Examples

```
EpiCompare::EpiCompare(peakfiles = list(CnT=CnT_H3K27ac, CnR=CnR_H3K27ac), genome_build = list(peakfiles="hg19", reference="hg38"), genome_build_output = "hg19", picard_files = picard_files, reference = list(ENCODE_H3K27ac=encode_H3K27ac), output_dir = tempdir())
```

## Evaluation signals

- The generated EpiCompare_report.html exists and contains all three sections: General Metrics, Peak Overlap, and Functional Annotation
- Peak counts and blacklist-overlap percentages match values obtained after filtering peaks against the ENCODE hg38 blacklist
- Report metrics and plots quantitatively match the published EpiArchives reference report 'atac_dnase_chip_example'
- Precision-recall plot against the H3K27ac ChIP-seq reference discriminates expected differences (e.g. ATAC-seq vs DNase-seq most evident in ChromHMM annotations and precision-recall)
- Output is a single interactive HTML report collating results for all query peak files, not per-sample separate reports

## Limitations

- Peaks in blacklisted regions and on non-standard chromosomes are removed prior to analysis, so counts differ from raw input files
- Documentation pertains to the development version; older Bioconductor versions may have different available functions and parameters
- Mixed genome builds across inputs must be declared explicitly (e.g. genome_build = list(peakfiles='hg19', reference='hg38')) or liftover/annotation may be wrong

## Evidence

- [abstract] One-line run with single HTML report: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] Three report sections: [evidence span withheld — anchor status: not_found]
- [readme] Blacklist filtering: [evidence span withheld — anchor status: external_doc]
- [readme] Genome build parameterization: [evidence span withheld — anchor status: external_doc]
- [intro] Benchmarking against gold standard: "it is important that we systematically compare these technologies and benchmark against a "gold-standard"."
