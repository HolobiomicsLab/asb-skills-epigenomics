---
name: epigenomic-peak-granges-manipulation
description: Use when peak files originate from different genome reference builds
  (e.g. hg19 and hg38 mixed inputs) or raw BED formats, and they must be imported
  as GRanges, coordinate-lifted, and standardized before EpiCompare quality-control
  and benchmarking.
when_to_use_negative:
- All input peak files already share the same genome build and are already GRanges
  or consistently formatted BED — no liftOver/build harmonization step is needed
- The goal is Picard duplication-rate summary import — EpiCompare::gather_files with
  type='picard' imports these as data frames, not as peak GRanges for coordinate manipulation
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: EpiCompare
  role: R package whose peak-import utilities, liftover_grlist and genome_build/genome_build_output
    parameters orchestrate GRanges import, build harmonization and blacklist filtering
  repo: https://github.com/neurogenomics/EpiCompare
- name: rtracklayer
  role: Backend used to import BED files as GRanges and to execute rtracklayer::liftOver
    across genome builds via chain files
- name: Bioconductor
  role: Ecosystem supplying the GRanges/rtracklayer infrastructure; EpiCompare is
    distributed via Bioconductor (>= v3.15)
  repo: https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
- name: R
  role: Language in which the GRanges manipulation and liftOver workflow is implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:ef72dfbc594ac6b352d8e235c5dfc72166e3d131e2ad4a3f3087da2497a54388
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
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# epigenomic-peak-granges-manipulation

## Summary

Import epigenomic peak files (BED, narrowPeak, .stringent.bed.gz) into R as GRanges objects, harmonize their genome builds via rtracklayer liftOver with UCSC over.chain files, and standardize them (drop blacklisted and non-standard chromosomes) so EpiCompare can compare CUT&Run/CUT&Tag/ChIP-seq peak sets in a single QC report.

## When to use

Use when peak files originate from different genome reference builds (e.g. hg19 and hg38 mixed inputs) or raw BED formats, and they must be imported as GRanges, coordinate-lifted, and standardized before EpiCompare quality-control and benchmarking.

## When NOT to use

- All input peak files already share the same genome build and are already GRanges or consistently formatted BED — no liftOver/build harmonization step is needed
- The goal is Picard duplication-rate summary import — EpiCompare::gather_files with type='picard' imports these as data frames, not as peak GRanges for coordinate manipulation

## Inputs

- Peak files in BED format (e.g. *_peaks.stringent.bed, narrowPeak) as file paths or directories
- Named list of peakfiles (e.g. CnT_H3K27ac, CnR_H3K27ac example GRanges)
- Reference peak file (e.g. ENCODE H3K27ac, accession ENCFF044JNJ)
- UCSC over.chain file (e.g. hg19ToHg38.over.chain) for liftOver
- genome_build specification mapping peakfiles/reference to hg19 or hg38

## Outputs

- GRanges objects with peak intervals harmonized to a single genome build (e.g. hg38)
- Lifted peak files exported as BED (e.g. CnR_H3K27ac_hg38.bed)
- Standardized peak sets (blacklisted and non-standard chromosomes removed) ready for EpiCompare QC

## How to apply

Load each peak file into R as a GRanges object using EpiCompare's import utilities (rtracklayer/BiocGenerics backend); EpiCompare::gather_files can batch-import a directory of 'peaks.stringent' BED files, and file paths to BED files are also accepted directly. When input builds differ from the requested output build (genome_build_output, e.g. 'hg19' vs 'hg38'), fetch the corresponding UCSC over.chain file (e.g. hg19ToHg38.over.chain), construct the chain list required by EpiCompare's liftover_grlist, and apply rtracklayer::liftOver to translate all peak intervals, dropping intervals with no mappable target. Prior to analysis, remove peaks overlapping ENCODE blacklist regions (hg19 or hg38) and peaks on non-standard chromosomes, as EpiCompare discards these from the files before QC. Export the standardized GRanges back to BED (e.g. CnR_H3K27ac_hg38.bed) so the harmonized files feed the General Metrics, Peak Overlap and Functional Annotation sections of the single HTML report. Declare builds explicitly, e.g. genome_build = list(peakfiles='hg19', reference='hg38') with genome_build_output set to the target build.

## Related tools

- **EpiCompare** (R package whose peak-import utilities, liftover_grlist and genome_build/genome_build_output parameters orchestrate GRanges import, build harmonization and blacklist filtering) — https://github.com/neurogenomics/EpiCompare
- **rtracklayer** (Backend used to import BED files as GRanges and to execute rtracklayer::liftOver across genome builds via chain files)
- **Bioconductor** (Ecosystem supplying the GRanges/rtracklayer infrastructure; EpiCompare is distributed via Bioconductor (>= v3.15)) — https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
- **R** (Language in which the GRanges manipulation and liftOver workflow is implemented)

## Examples

```
peakfiles <- EpiCompare::gather_files(dir = "path/to/peaks/", type = "peaks.stringent")
```

## Evaluation signals

- All output GRanges seqlevels conform to the declared genome_build_output build (e.g. hg38 chromosome naming) and differ from the input build coordinates
- Intervals with no mappable target in the chain file are dropped from the lifted GRanges, so the lifted set has equal or fewer peaks than the input
- No peaks on non-standard chromosomes and no blacklist-overlapping peaks remain prior to downstream EpiCompare analysis
- The exported BED (e.g. CnR_H3K27ac_hg38.bed) re-imports as a valid GRanges and can be run through EpiCompare() without genome-build mismatch errors
- Example example files (encode_H3K27ac, CnT_H3K27ac, CnR_H3K27ac) loaded via data() process through the same import path without errors

## Limitations

- LiftOver drops peak intervals with no mappable target region in the destination build, so lifted files may contain fewer peaks than the originals
- Documentation pertains to the development version of EpiCompare; older Bioconductor versions may have different available functions and parameters
- Input genome builds must be declared correctly (hg19 vs hg38); a mismatched genome_build declaration would silently corrupt coordinate translation

## Evidence

- [other] EpiCompare harmonizes genome builds using rtracklayer liftOver: [evidence span withheld — anchor status: not_found]
- [readme] EpiCompare::gather_files for peak import: [evidence span withheld — anchor status: external_doc]
- [readme] Blacklist and non-standard chromosome removal: [evidence span withheld — anchor status: external_doc]
- [abstract] Bioconductor R package for epigenomic QC: "EpiCompare, a Bioconductor (Huber et al., 2015) R package for the comparison and quality control of epigenomic data"
- [intro] hg19/hg38 blacklist availability: "It also contains human genome hg19 and hg38 blacklisted regions obtained from ENCODE."
