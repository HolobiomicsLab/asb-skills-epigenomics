---
name: peak-file-bed-import-export
description: Use when input consists of peak BED files (e.g. *_peaks.stringent.bed
  from CUT&Tag/CUT&Run/ChIP-seq) that must be loaded as GRanges for QC/comparison,
  or when harmonized/lifted intervals must be written back out to a BED file.
when_to_use_negative:
- Input is already a GRanges object or in-memory peak table — no BED import/export
  step is needed
- The goal is raw read alignment or peak calling; this skill only handles called peak
  intervals in BED format
- Files are Picard metrics or other non-interval tabular outputs — use the picard-type
  import path, not peak BED import
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: EpiCompare
  role: Provides peak-import utilities (gather_files, EpiCompare()) and liftover_grlist
    for importing BED peaks as GRanges and exporting harmonized files
  repo: https://github.com/neurogenomics/EpiCompare
- name: rtracklayer
  role: Backend for reading BED files into GRanges, running liftOver, and exporting
    GRanges back to BED format
- name: R
  role: Runtime environment for the Bioconductor-based import/export workflow
- name: Bioconductor
  role: Distribution channel providing rtracklayer and related GRanges infrastructure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:942c710bc2213a2a96ec36da1dad1abf186e7b64267028607dfa010c70504490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# peak-file-bed-import-export

## Summary

Import BED-format epigenomic peak files (e.g. CUT&Tag, CUT&Run, ChIP-seq peaks) into R as GRanges objects and export harmonized peak files back to BED, using EpiCompare's peak-import utilities backed by rtracklayer/BiocGenerics so the intervals can feed downstream QC checks.

## When to use

Use when input consists of peak BED files (e.g. *_peaks.stringent.bed from CUT&Tag/CUT&Run/ChIP-seq) that must be loaded as GRanges for QC/comparison, or when harmonized/lifted intervals must be written back out to a BED file.

## When NOT to use

- Input is already a GRanges object or in-memory peak table — no BED import/export step is needed
- The goal is raw read alignment or peak calling; this skill only handles called peak intervals in BED format
- Files are Picard metrics or other non-interval tabular outputs — use the picard-type import path, not peak BED import

## Inputs

- BED-format peak files (e.g. *_peaks.stringent.bed) from CUT&Tag, CUT&Run, TIP-seq or ChIP-seq experiments
- Picard summary output .txt files (importable via EpiCompare::gather_files type='picard')
- Named lists of peakfiles/reference passed to EpiCompare()
- UCSC chain file (e.g. hg19ToHg38.over.chain) for liftOver-based build conversion

## Outputs

- GRanges objects holding peak intervals per sample
- Exported BED files with harmonized coordinates (e.g. CnR_H3K27ac_hg38.bed)
- Peak files ready for EpiCompare QC (blacklist/non-standard chromosome removal, overlap, annotation)

## How to apply

Import each peak BED file as a GRanges object using EpiCompare's peak-import utilities (rtracklayer/BiocGenerics backend), either by passing file paths directly to EpiCompare() or by using EpiCompare::gather_files(dir=..., type='peaks.stringent') to discover and import files from a directory. Because EpiCompare removes peaks in blacklisted regions (ENCODE hg19/hg38 blacklists) and non-standard chromosomes prior to analysis, import retains genomic coordinates and metadata needed for those filters. For cross-genome-build harmonization, use the imported GRanges with rtracklayer::liftOver and a UCSC chain file (e.g. hg19ToHg38.over.chain), dropping intervals with no mappable target. Finally, export the resulting GRanges to a BED file (e.g. CnR_H3K27ac_hg38.bed) via rtracklayer's export so downstream EpiCompare QC sections (peak overlap, ChromHMM, ChIPseeker annotation) receive a consistent genome build.

## Related tools

- **EpiCompare** (Provides peak-import utilities (gather_files, EpiCompare()) and liftover_grlist for importing BED peaks as GRanges and exporting harmonized files) — https://github.com/neurogenomics/EpiCompare
- **rtracklayer** (Backend for reading BED files into GRanges, running liftOver, and exporting GRanges back to BED format)
- **R** (Runtime environment for the Bioconductor-based import/export workflow)
- **Bioconductor** (Distribution channel providing rtracklayer and related GRanges infrastructure)

## Examples

```
peakfiles <- EpiCompare::gather_files(dir = "path/to/peaks/", type = "peaks.stringent")
```

## Evaluation signals

- Imported object is a GRanges with seqnames, ranges, and strand populated from the BED columns
- After liftOver, intervals lacking a mappable target are dropped — coordinate ranges must all lie within the target build (e.g. hg38) chrom sizes
- Exported BED file round-trips: re-importing CnR_H3K27ac_hg38.bed reproduces the same GRanges coordinates
- Peaks in ENCODE blacklist regions (hg19/hg38) and non-standard chromosomes are flagged/removed before downstream QC per EpiCompare's stated behavior
- Downstream EpiCompare run with mixed builds (peakfiles='hg19', reference='hg38') completes only after the genome_build_output harmonization step succeeds

## Limitations

- Peaks in blacklisted regions and on non-standard chromosomes are removed prior to analysis, so exported files reflect filtered intervals
- liftOver drops intervals with no mappable target, so peak counts may decrease after build conversion
- Documentation in the README pertains to the development version; older Bioconductor versions may expose different functions/parameters

## Evidence

- [other] liftOver-based harmonization: [evidence span withheld — anchor status: not_found]
- [readme] gather_files import: "`EpiCompare::gather_files` is helpful for identifying and importing peak
or picard files."
- [readme] blacklist filtering: "*Note*: Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [abstract] package scope: "EpiCompare is available on Bioconductor (≥ v3.15): https://bioconductor.org/packages/release/bioc/html/EpiCompare.html"
- [readme] mixed genome builds: "genome_build = list(peakfiles="hg19",
                                           reference="hg38"),
                       genome_build_output = "hg19""
