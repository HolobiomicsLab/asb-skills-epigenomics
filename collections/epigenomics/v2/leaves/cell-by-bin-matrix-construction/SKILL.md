---
name: cell-by-bin-matrix-construction
description: Use when raw scATAC-seq fragment/BAM data must be converted into a snap-format
  cell-by-bin count matrix (e.g., at 5-kb resolution) as the starting feature representation
  for clustering or dimensionality reduction without population-level peak calling.
when_to_use_negative:
- Input is already a cell-by-peak or cell-by-bin feature table produced by CellRanger-ATAC
  or another pipeline — proceed directly to binarization and Jaccard kernel computation.
- The analysis goal requires peak-level resolution (e.g., motif or gene-activity analysis)
  rather than genome-wide binned clustering features.
- Working in R >= 3.6.0, since rhdf5 fails on that version for the original SnapATAC
  package.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: SnapTools
  role: Python module used to create the snap file from BAM/bed ('snap-pre') and add
    cell-by-bin count matrices at multiple resolutions ('snap-add-bmat')
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: R package providing addBmatToSnap to add the cell-by-bin matrix to the snap
    object for downstream binarization and Jaccard kernel computation
  repo: https://github.com/r3fang/SnapATAC
- name: SnapATAC2
  role: Successor package with blazingly fast BAM-to-fragment and count matrix generation
    tools for the same cell-by-bin construction step
  repo: https://github.com/scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:d1ad05cf227503a1a9cf6a742ce719c988896ca17b1b27c1bad4fe4a176cd567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapTools
  tools:
  - Python
  - Python 2.7
  - R
  - R 3.4.0
  - SnapATAC
  - SnapATAC2
  - SnapTools
  - cisTopic
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cell-by-bin-matrix-construction

## Summary

Construction of a cell-by-bin accessibility matrix from scATAC-seq fragments, where the genome is tiled into fixed-size bins (e.g., 5-kb) and each cell is represented by its fragment counts per bin, forming the raw feature space for SnapATAC's downstream Jaccard-kernel analysis.

## When to use

Use when raw scATAC-seq fragment/BAM data must be converted into a snap-format cell-by-bin count matrix (e.g., at 5-kb resolution) as the starting feature representation for clustering or dimensionality reduction without population-level peak calling.

## When NOT to use

- Input is already a cell-by-peak or cell-by-bin feature table produced by CellRanger-ATAC or another pipeline — proceed directly to binarization and Jaccard kernel computation.
- The analysis goal requires peak-level resolution (e.g., motif or gene-activity analysis) rather than genome-wide binned clustering features.
- Working in R >= 3.6.0, since rhdf5 fails on that version for the original SnapATAC package.

## Inputs

- Aligned BAM file of scATAC-seq paired-end reads (or FASTQ pairs to align first)
- Reference genome FASTA and chromosome size file (e.g., mm10.fa, mm10.chrom.size)
- Snap file (hdf5, version 4) with HD, BD and FM sessions

## Outputs

- Snap file with cell-by-bin count matrix added at chosen bin sizes (e.g., 5-kb and 10-kb) in the AM session
- Library QC summary file (demo.snap.qc) with fragment counts per barcode

## How to apply

Preprocess reads with SnapTools (demultiplex, align with BWA, filter MAPQ > 30, fragment length < 1000 bp, remove PCR duplicates) and create a snap file via 'snaptools snap-pre'. Then tile the genome into fixed-size bins — SnapATAC uses 5-kb bins as its default genome-wide resolution — and count fragments per cell per bin using 'snaptools snap-add-bmat' (CLI) or 'addBmatToSnap' (R), which appends the matrix into the AM session of the snap file. Multiple bin sizes (e.g., 5000 and 10000) can be stored in the same snap file. The rationale is that fixed genome-wide bins avoid the bias of predefined, population-level peak references, which underrepresent rare cell types; binarization (makeBinary with outlier.filter) is applied downstream, not at this step. QC the library via the generated .snap.qc file (usable fragments, unique fragments, chrM fragments) before proceeding.

## Related tools

- **SnapTools** (Python module used to create the snap file from BAM/bed ('snap-pre') and add cell-by-bin count matrices at multiple resolutions ('snap-add-bmat')) — https://github.com/r3fang/SnapTools
- **SnapATAC** (R package providing addBmatToSnap to add the cell-by-bin matrix to the snap object for downstream binarization and Jaccard kernel computation) — https://github.com/r3fang/SnapATAC
- **SnapATAC2** (Successor package with blazingly fast BAM-to-fragment and count matrix generation tools for the same cell-by-bin construction step) — https://github.com/scverse/SnapATAC2

## Examples

```
snaptools snap-add-bmat --snap-file=demo.snap --bin-size-list 5000 10000 --verbose=True
```

## Evaluation signals

- demo.snap.qc reports plausible counts: proper paired fragments dominate, usable fragments ≈ unique fragments, chrM fragments ≈ 0 (e.g., 539772 usable of 576676 total in the demo).
- snap-add-bmat reports successful addition of the AM session at each requested bin size; addBmatToSnap completes without error on R 3.4.x.
- The bin matrix dimensions equal number of barcodes × number of genome-wide bins at the chosen bin size (e.g., ~5-kb bins over mm10).
- Downstream binarization (makeBinary) yields per-cell binary vectors, and the Jaccard kernel computed from them produces leading eigenvectors that separate known PBMC populations.

## Limitations

- SnapATAC (v1) requires Python >= 2.7 & < 3.0 for SnapTools and R >= 3.4.0 & < 3.6.0 for SnapATAC because rhdf5 fails on R 3.6.
- At genome-wide 5-kb bin resolution the per-cell profile is extremely sparse (only 5–15% of peaks detected per cell), so binarization and depth normalization are mandatory downstream.
- The snap format stores multiple bin resolutions in one file; users must select the appropriate bin size before kernel computation.

## Evidence

- [readme] snap-add-bmat   Add cell x bin count matrix to snap file.: [evidence span withheld — anchor status: external_doc]
- [readme] three cell-by-bin matrices are created with bin size of 5,000 and 10,000: [evidence span withheld — anchor status: external_doc]
- [intro] SnapATAC first performs preprocessing of sequencing reads including demultiplexing, reads alignments and filtering, duplicate removal and barcode selection using SnapTools: "SnapATAC first performs preprocessing of sequencing reads including demultiplexing, reads alignments and filtering, duplicate removal and barcode selection using SnapTools"
- [full_text] addBmatToSnap at 5-kb bin resolution: [evidence span withheld — anchor status: not_found]
- [readme] We only keep the properly paired fragments whose MAPQ (--min-mapq) is greater than 30 with fragment length less than 1000bp (--max-flen).: [evidence span withheld — anchor status: external_doc]
