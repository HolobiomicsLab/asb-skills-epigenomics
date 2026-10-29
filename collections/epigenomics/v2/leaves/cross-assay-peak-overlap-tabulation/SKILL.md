---
name: cross-assay-peak-overlap-tabulation
description: Use when two or more peak files from different epigenomic assays or technologies
  (narrowPeak/BED, e.g. ENCODE ATAC-seq and DNase-seq) must be compared at the chromatin-state
  annotation level, as in EpiCompare's ChromHMM functional-annotation report section.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) or un-called alignments — EpiCompare operates
  on already-called peak files
- Only a single peak file with no second assay/condition to contrast — there is no
  cross-assay comparison to tabulate
- Non-human genomes (hg19/hg38) — the packaged ENCODE blacklists and ChromHMM annotations
  cover human builds only
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: Bioconductor R package that imports/standardizes peak files as GRanges, intersects
    them with ChromHMM annotations, tabulates per-state percentages, and renders the
    chromHMM heatmap in a single HTML report
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment in which the K562 ATAC-seq and DNase-seq GRanges peak
    objects are loaded and compared
- name: knitr
  role: Vignette/report engine that renders EpiCompare's HTML output
- name: rmarkdown
  role: Vignette engine used with BiocStyle to produce the report document
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:83632871f61dfd1693dda0760df6df171c9a6229480f7eb6c5e1cc2aec25149a
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cross-assay-peak-overlap-tabulation

## Summary

Quantify how peak sets from different chromatin-profiling assays (e.g. K562 ATAC-seq vs DNase-seq narrowPeak files) are distributed across ChromHMM chromatin states using EpiCompare, then contrast per-state overlap percentages between the assays.

## When to use

Use when two or more peak files from different epigenomic assays or technologies (narrowPeak/BED, e.g. ENCODE ATAC-seq and DNase-seq) must be compared at the chromatin-state annotation level, as in EpiCompare's ChromHMM functional-annotation report section.

## When NOT to use

- Input is raw sequencing data (FASTQ) or un-called alignments — EpiCompare operates on already-called peak files
- Only a single peak file with no second assay/condition to contrast — there is no cross-assay comparison to tabulate
- Non-human genomes (hg19/hg38) — the packaged ENCODE blacklists and ChromHMM annotations cover human builds only

## Inputs

- Peak files in narrowPeak/BED format (one per assay/technology, e.g. K562 ATAC-seq and DNase-seq; importable via EpiCompare::gather_files or as GRanges)
- ChromHMM chromatin-state annotation for the relevant cell line (e.g. K562)
- Genome build specification (hg19 or hg38) for peakfiles and reference
- Optional Picard summary output and reference peak sets for extended QC context

## Outputs

- Interactive HTML report section containing the ChromHMM annotation heatmap of peak states
- Tabulation of the percentage of peaks in each peak file overlapping each ChromHMM state
- Per-assay comparison of chromatin-state distributions (basis for Fig. 1d-style differential annotation findings)

## How to apply

Import each assay's peak file (e.g. K562 ATAC-seq and DNase-seq narrowPeak, ENCODE accession ENCFF044JNJ for DNase-seq) into R and standardize as GRanges with EpiCompare; peaks in ENCODE blacklist regions (hg19/hg38) and on non-standard chromosomes are removed first. Supply the ChromHMM annotation matching the cell line (K562) and the relevant genome build, so each peak set is intersected with the chromatin-state segments and the percentage of peaks overlapping each state is tabulated. Render EpiCompare's chromHMM annotation heatmap for the peak files and contrast the per-state overlap percentages between assays to identify annotation differences (e.g. the ATAC-seq vs DNase-seq difference reported in Fig. 1d). EpiCompare collates all of this into a single interactive HTML report run with essentially one call.

## Related tools

- **EpiCompare** (Bioconductor R package that imports/standardizes peak files as GRanges, intersects them with ChromHMM annotations, tabulates per-state percentages, and renders the chromHMM heatmap in a single HTML report) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment in which the K562 ATAC-seq and DNase-seq GRanges peak objects are loaded and compared)
- **knitr** (Vignette/report engine that renders EpiCompare's HTML output)
- **rmarkdown** (Vignette engine used with BiocStyle to produce the report document)

## Examples

```
EpiCompare::EpiCompare(peakfiles=list(ATAC=atac_gr, DNASE=dnase_gr), genome_build='hg38', chromHMM_annotation='K562', output_dir='report')
```

## Evaluation signals

- The generated ChromHMM heatmap and per-state tabulation show a differential chromatin-state distribution between the ATAC-seq and DNase-seq peak files, consistent with the Fig. 1d annotation difference
- Peaks in ENCODE blacklist regions and on non-standard chromosomes are absent from the analysis (they are removed prior to intersection)
- Output is a single interactive HTML report containing the General Metrics, Peak Overlap, and Functional Annotation (ChromHMM) sections
- Per-state percentages are computed as the fraction of each peak file's peaks overlapping each ChromHMM state, allowing direct ATAC-seq vs DNase-seq contrast
- Reproducible code and hosted reports are available via the EpiArchives archive (e.g. the atac_dnase_example report and its EpiCompare_code.R)

## Limitations

- Peaks in blacklisted regions and non-standard chromosomes are discarded before ChromHMM intersection, so tabulated percentages exclude these regions
- Documentation differs between EpiCompare versions; older Bioconductor versions may have different functions/parameters than the development-version README
- Human hg19/hg38 scope only — the built-in ENCODE blacklists and ChromHMM cell-line annotations do not cover other genomes

## Evidence

- [readme] Functional annotation (ChromHMM, ChIPseeker and enrichment analysis) of peaks: "**Functional Annotation**: Functional annotation (ChromHMM, ChIPseeker and enrichment analysis) of peaks. Also includes peak enrichment around Transcription Start Site."
- [readme] Blacklist filtering before analysis: [evidence span withheld — anchor status: external_doc]
- [intro] ENCODE DNase-seq accession used: "ENCODE (data accession: ENCFF044JNJ)"
- [abstract] Single-line execution producing one HTML report: "The package requires minimal input from users, can be run with just one line of code and provides all results of the analysis in a single interactive HTML report."
- [readme] Peak overlap statistics definition: [evidence span withheld — anchor status: external_doc]
