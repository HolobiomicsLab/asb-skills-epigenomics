---
name: snap-file-preprocessing
description: Use when starting from raw scATAC-seq FASTQ or aligned BAM files and
  you need to build a snap file (v4, HDF5) with fragment, barcode, and cell-by-bin
  matrices as input to the SnapATAC R pipeline.
when_to_use_negative:
- Input is already a snap file or a processed cell-by-feature matrix — proceed directly
  to SnapATAC R functions (makeBinary, runDimReduct) instead
- You are using SnapATAC2 (scverse), which takes fragment files directly and does
  not need the snap format
- Data are single-cell RNA-seq or other non-ATAC modalities — the snap format is specific
  to single nucleus ATAC-seq profiles
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: SnapTools
  role: Python module that performs demultiplexing, alignment, filtering, duplicate
    removal, barcode selection, and snap-file creation (snap-pre, snap-add-bmat/pmat/gmat)
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: Downstream R package that reads the snap file and performs binary matrix conversion,
    dimensionality reduction, and Leiden clustering
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:8470a5d249cd0d33e0bcb1b1113376d6fc2390c86469e06bba7661048595a772
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapTools
  tools:
  - R
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# snap-file-preprocessing

## Summary

Convert raw scATAC-seq FASTQ/BAM data into the hierarchically structured snap (Single Nucleus Accessibility Profile) HDF5 format using SnapTools, producing cell-by-bin, cell-by-peak, and cell-by-gene matrices that SnapATAC consumes for downstream clustering.

## When to use

Use when starting from raw scATAC-seq FASTQ or aligned BAM files and you need to build a snap file (v4, HDF5) with fragment, barcode, and cell-by-bin matrices as input to the SnapATAC R pipeline.

## When NOT to use

- Input is already a snap file or a processed cell-by-feature matrix — proceed directly to SnapATAC R functions (makeBinary, runDimReduct) instead
- You are using SnapATAC2 (scverse), which takes fragment files directly and does not need the snap format
- Data are single-cell RNA-seq or other non-ATAC modalities — the snap format is specific to single nucleus ATAC-seq profiles

## Inputs

- Paired-end scATAC-seq FASTQ files (R1/R2)
- Reference genome FASTA (e.g. mm10.fa) and chrom.sizes file
- Optionally an already-aligned BAM file (snap-pre accepts BAM or BED input)

## Outputs

- Version-4 snap HDF5 file (.snap) with HD, BD, AM, PM, GM, and FM sessions
- Library QC report (.snap.qc) with barcode/fragment/alignment counts
- Cell-by-bin accessibility matrices at multiple resolutions (e.g. 5 kb, 10 kb) stored in the AM session

## How to apply

Run the SnapTools command-line suite: index the reference genome (e.g. mm10 with BWA) if needed, align paired-end reads with 'snaptools align-paired-end' (sorted by read name with --if-sort so barcode-grouped fragments can be de-duplicated), then create the snap file with 'snaptools snap-pre' using quality cutoffs of MAPQ >= 30 (--min-mapq=30), fragment length <= 1000 bp (--max-flen=1000), and a minimum coverage filter (--min-cov=100). snap-pre emits the .snap file plus a .snap.qc text file of library-level QC metrics (unique barcodes, usable/unique fragments, chrM fraction). Finally add cell-by-bin matrices at chosen resolutions (e.g. 5000 and 10000 bp) via 'snaptools snap-add-bmat', and optionally peak and gene matrices with snap-add-pmat / snap-add-gmat; the snap file stores sessions HD, BD, AM, PM, GM, and FM. SnapTools requires Python (recommended 2.7) with pysam, h5py, numpy, and pybedtools.

## Related tools

- **SnapTools** (Python module that performs demultiplexing, alignment, filtering, duplicate removal, barcode selection, and snap-file creation (snap-pre, snap-add-bmat/pmat/gmat)) — https://github.com/r3fang/SnapTools
- **SnapATAC** (Downstream R package that reads the snap file and performs binary matrix conversion, dimensionality reduction, and Leiden clustering) — https://github.com/r3fang/SnapATAC

## Examples

```
snaptools snap-pre --input-file=demo.bam --output-snap=demo.snap --genome-name=mm10 --genome-size=mm10.chrom.size --min-mapq=30 --max-flen=1000 --overwrite=True --min-cov=100
```

## Evaluation signals

- The .snap.qc report exists and reports plausible counts (e.g. total fragments, unique fragments, proper pairs, chrM fragments) for the library
- snap-pre runs with --min-mapq=30 and --max-flen=1000 and reports usable fragments for all intended barcodes
- snap-add-bmat successfully writes cell-by-bin matrices into the AM session at the requested bin sizes (e.g. 5000, 10000) without creating a separate file
- The resulting snap file opens in SnapATAC R (R >= 3.4.0 & < 3.6.0) and addBmatToSnap/makeBinary succeed, confirming HDF5 (rhdf5) session integrity

## Limitations

- SnapATAC R requires R >= 3.4.0 and < 3.6.0 because R 3.6 breaks the rhdf5 dependency; SnapTools recommends Python 2.7 (version 1.4.+ supports both python2 and python3)
- Alignment requires an indexed reference genome (BWA) and read-name sorting so fragments from the same barcode can be grouped and PCR duplicates removed
- The snap format is for single nucleus ATAC-seq only; SnapATAC2 has superseded SnapATAC 1.x for newer workflows

## Evidence

- [intro] SnapATAC first performs preprocessing of sequencing reads including demultiplexing, reads alignments and filtering, duplicate removal and barcode selection using SnapTools: "SnapATAC first performs preprocessing of sequencing reads including demultiplexing, reads alignments and filtering, duplicate removal and barcode selection using SnapTools"
- [readme] snap file format definition: [evidence span withheld — anchor status: external_doc]
- [readme] snap-pre filtering parameters: [evidence span withheld — anchor status: not_found]
- [readme] version requirements: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [readme] adding cell-by-bin matrices: [evidence span withheld — anchor status: external_doc]
