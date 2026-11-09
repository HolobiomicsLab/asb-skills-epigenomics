---
name: epigenomic-peak-file-granges-handling
description: Use when peak files from epigenomic assays (narrowPeak/BED, e.g., ENCODE
  ENCFF044JNJ K562 DNase-seq) need to be loaded into R and standardized as GRanges
  with consistent genome build and blacklist filtering prior to overlap or ChromHMM
  annotation analysis.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) rather than called peak files — peak calling
  must happen first
- Input is already a feature/segment table (e.g., ChromHMM chromatin-state segments)
  rather than interval peak files
- The goal is downstream functional annotation (ChIPseeker, enrichment) on peaks that
  are already standardized GRanges
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: EpiCompare
  role: Imports peak files as GRanges, applies ENCODE blacklist/non-standard-chromosome
    filtering, and harmonizes genome builds via genome_build/genome_build_output parameters
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment in which EpiCompare (Bioconductor >= v3.15) is installed
    and GRanges standardization is executed
  repo: https://github.com/neurogenomics/EpiCompare
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:1979d96085167b6288e221ad82b947f5b5218a26e2f6c17ad8a3bb0cd422e0b2
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# epigenomic-peak-file-granges-handling

## Summary

Import and standardize epigenomic peak files (e.g., narrowPeak BED from ATAC-seq or DNase-seq) as GRanges objects in R using EpiCompare, removing ENCODE blacklist and non-standard chromosome peaks and harmonizing genome builds before downstream comparison.

## When to use

Use when peak files from epigenomic assays (narrowPeak/BED, e.g., ENCODE ENCFF044JNJ K562 DNase-seq) need to be loaded into R and standardized as GRanges with consistent genome build and blacklist filtering prior to overlap or ChromHMM annotation analysis.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) rather than called peak files — peak calling must happen first
- Input is already a feature/segment table (e.g., ChromHMM chromatin-state segments) rather than interval peak files
- The goal is downstream functional annotation (ChIPseeker, enrichment) on peaks that are already standardized GRanges

## Inputs

- narrowPeak / BED peak files (e.g., K562 ATAC-seq and DNase-seq peaks, ENCODE accession ENCFF044JNJ)
- Named list of peakfile paths or GRanges objects
- Genome build labels (hg19 or hg38) per peakfile and reference
- Optional Picard summary output .txt files

## Outputs

- Standardized GRanges peak objects in a consistent genome build with blacklist and non-standard-chromosome peaks removed
- Named peakfile list ready for EpiCompare overlap/ChromHMM/ChIPseeker analysis sections of the HTML report

## How to apply

Load EpiCompare from Bioconductor (>= v3.15) and import peak files either from BED paths supplied directly or via EpiCompare::gather_files (e.g., type='peaks.stringent'), producing a named list of GRanges objects. Specify genome builds per input with the genome_build parameter (e.g., list(peakfiles='hg19', reference='hg38')) and set genome_build_output to a single build so all peak sets are harmonized. EpiCompare automatically removes peaks in ENCODE blacklist regions (hg19/hg38 blacklist data ship with the package) and on non-standard chromosomes before analysis. Files may be supplied as paths or pre-loaded GRanges; a Picard summary can be attached in parallel for duplication-rate metrics. Only after this standardization should peak sets be passed to overlap, correlation, or ChromHMM annotation functions.

## Related tools

- **EpiCompare** (Imports peak files as GRanges, applies ENCODE blacklist/non-standard-chromosome filtering, and harmonizes genome builds via genome_build/genome_build_output parameters) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment in which EpiCompare (Bioconductor >= v3.15) is installed and GRanges standardization is executed) — https://github.com/neurogenomics/EpiCompare

## Examples

```
peakfiles <- EpiCompare::gather_files(dir = "path/to/peaks/", type = "peaks.stringent")
```

## Evaluation signals

- All peak inputs are converted to GRanges objects assembled in a named list (e.g., list('CnT'=..., 'CnR'=...))
- Peaks overlapping ENCODE blacklist regions and on non-standard chromosomes are absent from the standardized objects
- genome_build_output forces a single reference build across peakfiles and reference sets (e.g., all hg19)
- Standardized peak sets feed the General Metrics section (blacklisted/non-standard peak percentages, peak widths) without import errors
- Downstream ChromHMM state-overlap percentages (as in the K562 ATAC-seq vs DNase-seq Fig. 1d comparison) can be tabulated from the standardized GRanges

## Limitations

- Blacklist and non-standard-chromosome removal is applied automatically prior to analysis, so peak counts differ from the raw input files
- Genome build mismatches between peakfiles and reference must be declared explicitly via the genome_build list, or liftover/harmonization assumptions may be wrong
- Documentation differs between release and development versions of EpiCompare (e.g., available functions and parameters)

## Evidence

- [readme] gather_files imports BED as GRanges: [evidence span withheld — anchor status: external_doc]
- [readme] Blacklist filtering prior to analysis: [evidence span withheld — anchor status: external_doc]
- [intro] hg19/hg38 blacklist data shipped with the package: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
- [readme] Alternative path-based input of BED files: [evidence span withheld — anchor status: external_doc]
