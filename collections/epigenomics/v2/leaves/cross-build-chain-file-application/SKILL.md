---
name: cross-build-chain-file-application
description: Use when input peak files (e.g. BED files from CUT&Run, CUT&Tag or ChIP-sek
  studies) carry coordinates from mixed genome builds (hg19 vs hg38) and must be harmonized
  to one build before overlap/QC analysis.
when_to_use_negative:
- All peak files are already annotated with the same genome build as genome_build_output,
  so no liftOver is invoked
- Input is not genomic interval data (e.g. Picard summary txt files imported as data
  frames), which contain no coordinates to translate
- The target build has no UCSC chain available for the source build (e.g. unchained
  non-model assemblies)
tools:
- name: EpiCompare
  role: Orchestrates genome-build harmonization via liftover_grlist before QC and
    benchmarking of peak files
  repo: https://github.com/neurogenomics/EpiCompare
- name: rtracklayer
  role: Provides the liftOver function and chain-file import backend used to translate
    GRanges coordinates across builds
- name: Bioconductor
  role: Distribution platform for EpiCompare (≥ v3.15) and the rtracklayer/BiocGenerics
    import backend
  repo: https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:8e87ca79aa2ea9583130a353a30e42b1f707d1c10d2327f872083534a541156b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  - rtracklayer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-build-chain-file-application

## Summary

Harmonizes epigenomic peak files generated against different genome reference builds by applying a UCSC chain file via rtracklayer::liftOver (wrapped by EpiCompare's liftover_grlist), so that hg19- and hg38-derived peak intervals can be compared in a single EpiCompare QC report.

## When to use

Use when input peak files (e.g. BED files from CUT&Run, CUT&Tag or ChIP-sek studies) carry coordinates from mixed genome builds (hg19 vs hg38) and must be harmonized to one build before overlap/QC analysis.

## When NOT to use

- All peak files are already annotated with the same genome build as genome_build_output, so no liftOver is invoked
- Input is not genomic interval data (e.g. Picard summary txt files imported as data frames), which contain no coordinates to translate
- The target build has no UCSC chain available for the source build (e.g. unchained non-model assemblies)

## Inputs

- Peak files as BED paths or GRanges objects (e.g. CnR_H3K27ac peak BED file from CUT&Run H3K27ac)
- Genome-build annotations per file (e.g. hg19, hg38) supplied via the genome_build parameter
- UCSC chain file for the source-to-target build conversion (e.g. hg19ToHg38.over.chain)
- ENCODE blacklist regions for hg19 and hg38 (bundled with EpiCompare)

## Outputs

- Lifted GRanges object with all peak intervals in the target build coordinates
- Converted peak file in BED format (e.g. CnR_H3K27ac_hg38.bed) for downstream EpiCompare QC checks
- Harmonized inputs feeding the single interactive HTML QC report (General Metrics, Peak Overlap, Functional Annotation sections)

## How to apply

Load each peak BED file into R as a GRanges object using EpiCompare's peak-import utilities (rtracklayer/BiocGenerics backend). Determine each file's genome build and pass it via the genome_build parameter (e.g. list(peakfiles="hg19", reference="hg38")) with genome_build_output set to the target build. Obtain the matching UCSC chain file (e.g. hg19ToHg38.over.chain), construct the chain list as required by liftover_grlist, and apply rtracklayer::liftOver to translate every peak interval to the target build, dropping intervals with no mappable target. Because EpiCompare removes peaks in ENCODE blacklist regions and non-standard chromosomes prior to analysis, liftover should run before downstream QC checks; the lifted GRanges can be exported as a BED file (e.g. CnR_H3K27ac_hg38.bed) for the report.

## Related tools

- **EpiCompare** (Orchestrates genome-build harmonization via liftover_grlist before QC and benchmarking of peak files) — https://github.com/neurogenomics/EpiCompare
- **rtracklayer** (Provides the liftOver function and chain-file import backend used to translate GRanges coordinates across builds)
- **Bioconductor** (Distribution platform for EpiCompare (≥ v3.15) and the rtracklayer/BiocGenerics import backend) — https://bioconductor.org/packages/release/bioc/html/EpiCompare.html

## Examples

```
EpiCompare::EpiCompare(peakfiles = peakfiles, genome_build = list(peakfiles="hg19", reference="hg38"), genome_build_output = "hg19", run_all = TRUE, output_dir = tempdir())
```

## Evaluation signals

- Chain file direction matches the declared source and target builds (e.g. hg19ToHg38 for hg19→hg38 liftover)
- After liftover, GRanges seqnames/coordinates lie within the target build's ranges and no source-build contigs remain
- Intervals with no mappable chain target are dropped rather than silently mis-mapped
- EpiCompare call declares consistent genome_build and genome_build_output values so the report reflects the harmonized build
- Downstream report shows blacklist/non-standard-chromosome filtering applied to the lifted files, not the originals

## Limitations

- Peaks falling in regions unmappable by the chain file are lost during liftover, shrinking the peak set
- Peaks in ENCODE blacklisted regions and non-standard chromosomes are removed prior to analysis, so lifted coordinates outside standard chromosomes never reach the report
- Documentation pertains to the development version (1.15.1); older EpiCompare versions may expose different parameters such as genome_build/liftover handling

## Evidence

- [other] EpiCompare harmonizes genome builds by using the rtracklayer liftOver function, invoked when needed, to translate the genomic coordinates of peak files across genome builds.: [evidence span withheld — anchor status: not_found]
- [readme] genome_build list and output build declaration in EpiCompare() usage: "genome_build = list(peakfiles="hg19", reference="hg38"),
                       genome_build_output = "hg19""
- [readme] Pre-analysis filtering of blacklisted and non-standard peaks: "Peaks located in blacklisted regions and non-standard
chromosomes are removed from the files prior to analysis."
- [readme] EpiCompare accepts BED file paths as peak input: [evidence span withheld — anchor status: external_doc]
