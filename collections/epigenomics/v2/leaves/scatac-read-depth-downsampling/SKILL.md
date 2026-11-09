---
name: scatac-read-depth-downsampling
description: Use when benchmarking how a scATAC-seq analysis method's cell-type assignments
  hold up at reduced sequencing depth, or when compiling runtime-vs-cell-count and
  reproducibility tables (e.g., on the ~81k-cell Cusanovich 13-mouse-tissue atlas).
when_to_use_negative:
- Input is already a feature matrix (cell-by-peak or cell-by-gene counts) rather than
  raw fragments/reads — depth reduction cannot be applied meaningfully post-quantification
- The goal is accuracy benchmarking on full-depth data rather than assessing robustness
  to reduced sequencing depth
- Working in SnapATAC2 (Python/Rust), which is designed to scale to millions of cells
  and uses its own matrix-free spectral embedding pipeline instead of the SnapTools/snap-file
  workflow
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SnapTools
  role: Preprocesses reads (demultiplexing, alignment, duplicate removal, barcode
    selection), creates snap files and cell-by-bin matrices, and performs the read-depth
    downsampling step in the reproducibility benchmark
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: Re-runs the full analysis on downsampled data — runNormJaccard, runDimReduct
    and runCluster (Leiden) with a fixed seed — to score clustering reproducibility
    against full-depth results
  repo: https://github.com/r3fang/SnapATAC
- name: cisTopic
  role: Alternative method re-run on the downsampled profiles with default topic-model
    settings as a reproducibility comparison baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:048a22ac945671720391acb409005e492ab71882eead60155cb9420c72fa7cf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapTools
  tools:
  - Python
  - R
  - SnapATAC
  - SnapTools
  - cisTopic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scatac-read-depth-downsampling

## Summary

Downsamples scATAC-seq reads to a reduced sequencing depth (e.g., 10% of the original, a 90% reduction) with SnapTools so that clustering reproducibility of a method can be scored against its full-depth results, as done in the SnapATAC scalability/reproducibility benchmark.

## When to use

Use when benchmarking how a scATAC-seq analysis method's cell-type assignments hold up at reduced sequencing depth, or when compiling runtime-vs-cell-count and reproducibility tables (e.g., on the ~81k-cell Cusanovich 13-mouse-tissue atlas).

## When NOT to use

- Input is already a feature matrix (cell-by-peak or cell-by-gene counts) rather than raw fragments/reads — depth reduction cannot be applied meaningfully post-quantification
- The goal is accuracy benchmarking on full-depth data rather than assessing robustness to reduced sequencing depth
- Working in SnapATAC2 (Python/Rust), which is designed to scale to millions of cells and uses its own matrix-free spectral embedding pipeline instead of the SnapTools/snap-file workflow

## Inputs

- Full-depth scATAC-seq snap files (hdf5 v4: HD/AM/PM/GM/BD/FM sessions) for the mouse atlas (GEO GSE100033, ~81,000 high-quality cells)
- Fragment/BAM files with per-cell barcodes for read downsampling
- Full-depth reference cluster/cell-type assignments from the original pipeline (Leiden clusters at chosen resolution)

## Outputs

- Downsampled (10%-depth) snap files with regenerated cell-by-bin matrices
- Per-method runtime-vs-cell-number table and clustering-reproducibility scores (benchmark_results.csv)
- Comparison of cell-type assignment consistency between full-depth and downsampled runs

## How to apply

Start from the preprocessed snap files of the full-depth dataset (e.g., ~81,000 high-quality cells from Cusanovich et al. mouse tissues, GSE100033), produced by SnapTools with fragment filtering at MAPQ > 30 and fragment length < 1000 bp. Downsample each library's reads to 10% of the original depth (90% reduction) using SnapTools, then rebuild the cell-by-bin matrices (e.g., snap-add-bmat at 5,000/10,000 bp bins). For each downsampled replicate, re-run the method's full pipeline — normalized Jaccard similarity (runNormJaccard), dimensionality reduction (runDimReduct for SnapATAC; runLSA for LSA; default topic-model settings for cisTopic) — and re-cluster with the Leiden algorithm via runCluster using a fixed seed. Score reproducibility as the consistency of cell-type assignments between the downsampled runs and the full-depth runs, and compile per-method, per-depth runtime and reproducibility values into a benchmark table (benchmark_results.csv). Keeping seed.use, pc.num and bin resolution matched across methods is essential so differences reflect depth robustness rather than stochasticity.

## Related tools

- **SnapTools** (Preprocesses reads (demultiplexing, alignment, duplicate removal, barcode selection), creates snap files and cell-by-bin matrices, and performs the read-depth downsampling step in the reproducibility benchmark) — https://github.com/r3fang/SnapTools
- **SnapATAC** (Re-runs the full analysis on downsampled data — runNormJaccard, runDimReduct and runCluster (Leiden) with a fixed seed — to score clustering reproducibility against full-depth results) — https://github.com/r3fang/SnapATAC
- **cisTopic** (Alternative method re-run on the downsampled profiles with default topic-model settings as a reproducibility comparison baseline)

## Evaluation signals

- snap QC report from snap-pre confirms the downsampled library's usable/unique fragment counts are scaled to ~10% of the full-depth values (e.g., TN/UQ rows in demo.snap.qc)
- Downsampled-vs-full-depth cell-type assignment consistency is computed with the same fixed seed.use and resolution as the full-depth run, so rerunning the pipeline reproduces the same clusters
- Per-method runtime and reproducibility values are logged for every depth point and compiled into benchmark_results.csv without missing cells/subsets (20k/40k/60k/80k)
- Fragment-level QC still passes after downsampling: MAPQ > 30, fragment length < 1000 bp, proper-paired reads only

## Limitations

- Version constraints: SnapTools runs on Python 2.7 (>=2.7 & <3.0 recommended) and SnapATAC on R >= 3.4.0 & < 3.6.0 because R 3.6 breaks the rhdf5 package
- Downsampling assesses depth robustness of cell-type assignments but does not directly test accuracy on rare populations; the paper notes rare-population signals can be underrepresented
- SnapATAC (v1) is superseded by SnapATAC2, which scales to more than 10 million cells and implements a different matrix-free spectral embedding algorithm
- Reproducibility scores depend on matched settings (pc.num, seed.use, resolution); unmatched parameters confound the depth effect

## Evidence

- [other] Downsample sequencing reads to 10% of the original depth (90% reduction) with SnapTools for each method's pipeline.: [evidence span withheld — anchor status: not_found]
- [other] Re-cluster with Leiden and score reproducibility against full-depth runs.: [evidence span withheld — anchor status: not_found]
- [intro] Reproducibility is a claimed differentiator of SnapATAC.: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [readme] SnapTools fragment filtering thresholds applied before and after downsampling.: [evidence span withheld — anchor status: external_doc]
- [methods] SnapTools preprocesses reads into snap files used for downsampling.: "SnapTools - a python module for pre-processing and working with [snap](https://github.com/r3fang/SnapATAC/wiki/FAQs) file."
