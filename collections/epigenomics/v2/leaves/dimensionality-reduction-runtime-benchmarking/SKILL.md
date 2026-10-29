---
name: dimensionality-reduction-runtime-benchmarking
description: Use when evaluating how the CPU runtime of dimensionality reduction for
  single-cell epigenomic data scales with cell number, or when comparing scalability/reproducibility
  of SnapATAC vs LSA vs cisTopic on cell-by-bin matrices.
when_to_use_negative:
- When the analysis goal is accuracy/cluster-quality benchmarking on a single small
  dataset rather than runtime scaling with cell number
- When the method under test cannot accept genome-wide cell-by-bin matrices and requires
  pre-called peaks or a topic-model input the SnapATAC pipeline does not provide
- When data are scRNA-seq or another modality — this benchmark is specific to scATAC-seq
  accessibility profiles
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: SnapATAC
  role: R package providing runNormJaccard, runDimReduct, runLSA and runCluster; the
    method whose runtime and reproducibility is benchmarked (plus its LSA variant)
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module for preprocessing reads into snap-format cell-by-bin matrices
    (snap-pre, snap-add-bmat) and downsampling reads to 10% depth
  repo: https://github.com/r3fang/SnapTools
- name: cisTopic
  role: Baseline topic-model method for dimensionality reduction, run with default
    settings on the same subsets to compare runtime
- name: R 3.4.0
  role: Runtime environment required for SnapATAC (R >= 3.4.0 & < 3.6.0 because 3.6
    breaks the rhdf5 package)
- name: Python 2.7
  role: Runtime environment for SnapTools preprocessing (>= 2.7 & < 3.0 recommended)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:78e1a64c935ae45068692666ec6334015213f673ed55d4c8469b36bf338e3199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python
  - Python 2.7
  - R
  - R 3.4.0
  - SnapATAC
  - SnapTools
  - cisTopic
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Dimensionality reduction runtime benchmarking

## Summary

Benchmark CPU running time of scATAC-seq dimensionality reduction methods (SnapATAC, LSA, cisTopic) as cell count scales from 20k to 1 million, using the Cusanovich 13-mouse-tissue atlas (~81,000 cells, GEO GSE100033) to quantify linear-scaling and reproducibility advantages of SnapATAC.

## When to use

Use when evaluating how the CPU runtime of dimensionality reduction for single-cell epigenomic data scales with cell number, or when comparing scalability/reproducibility of SnapATAC vs LSA vs cisTopic on cell-by-bin matrices.

## When NOT to use

- When the analysis goal is accuracy/cluster-quality benchmarking on a single small dataset rather than runtime scaling with cell number
- When the method under test cannot accept genome-wide cell-by-bin matrices and requires pre-called peaks or a topic-model input the SnapATAC pipeline does not provide
- When data are scRNA-seq or another modality — this benchmark is specific to scATAC-seq accessibility profiles

## Inputs

- scATAC-seq FASTQ/BAM files for the Cusanovich 13-mouse-tissue dataset (GEO GSE100033, ~81,000 cells)
- Snap-format (hdf5) cell-by-bin accessibility matrices at 5,000 bp resolution
- Random cell subsamples of 20k, 40k, 60k, 80k cells and a 1M-cell Nyström-simulated dataset
- Downsampled (10% depth) read profiles for reproducibility testing

## Outputs

- benchmark_results.csv with CPU runtime per method (SnapATAC, LSA, cisTopic) per cell count
- Normalized Jaccard similarity matrices per subset
- Leiden clustering assignments (fixed seed/resolution) at full vs downsampled depth
- Cell-type assignment consistency/reproducibility scores

## How to apply

Preprocess the Cusanovich et al. scATAC-seq dataset (GEO GSE100033, ~81,000 high-quality cells) with SnapTools into snap-format cell-by-bin matrices (e.g. 5,000 bp bins via snap-add-bmat). Generate random subsamples of 20k, 40k, 60k and 80k cells plus a simulated 1M-cell dataset using SnapATAC's ensemble Nyström approximation. For each subset, compute the normalized Jaccard similarity with runNormJaccard and run SnapATAC dimensionality reduction via runDimReduct(obj, pc.num, input.mat='jmat', method, center, scale, seed.use, maxit) with a fixed seed.use and a matched number of eigenvectors, logging CPU runtime. Run LSA via runLSA(obj, input.mat, pc.num, logTF, scale.factor, min.cell, seed.use) and cisTopic with its default topic-model settings on identical subsets, then compile a runtime-versus-cell-number table. Assess reproducibility by downsampling sequencing reads to 10% of original depth with SnapTools, re-clustering with runCluster (Leiden, fixed seed and resolution) and scoring consistency of cell-type assignments against full-depth runs.

## Related tools

- **SnapATAC** (R package providing runNormJaccard, runDimReduct, runLSA and runCluster; the method whose runtime and reproducibility is benchmarked (plus its LSA variant)) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module for preprocessing reads into snap-format cell-by-bin matrices (snap-pre, snap-add-bmat) and downsampling reads to 10% depth) — https://github.com/r3fang/SnapTools
- **cisTopic** (Baseline topic-model method for dimensionality reduction, run with default settings on the same subsets to compare runtime)
- **R 3.4.0** (Runtime environment required for SnapATAC (R >= 3.4.0 & < 3.6.0 because 3.6 breaks the rhdf5 package))
- **Python 2.7** (Runtime environment for SnapTools preprocessing (>= 2.7 & < 3.0 recommended))

## Examples

```
runDimReduct(obj, pc.num=15, input.mat="jmat", method="eigen», center=TRUE, scale=FALSE, seed.use=10); runLSA(obj, input.mat="pmat", pc.num=15, logTF=TRUE, scale.factor=100000, seed.use=10)
```

## Evaluation signals

- Runtime-versus-cell-number plot for SnapATAC should be approximately linear with a lower slope than LSA and cisTopic
- On 100k cells, SnapATAC should be ~10x faster than LSA and >100x faster than cisTopic
- SnapATAC should complete dimensionality reduction on 1M simulated cells (ensemble Nyström) on regular hardware while cisTopic/LSA do not scale
- Reproducibility: cell-type assignments from 10%-depth downsampled runs should match full-depth Leiden cluster labels at high consistency (same fixed seed.use and resolution)
- All benchmarked methods must use the same cell subsets, matched number of eigenvectors/topics, and fixed seed.use so runtimes are directly comparable

## Limitations

- Python version constraint: SnapTools recommended for Python >= 2.7 & < 3.0; SnapATAC requires R >= 3.4.0 & < 3.6.0 because R 3.6 does not work for the rhdf5 package
- Scalability beyond ~1M cells relies on the ensemble Nyström approximation of the Jaccard kernel rather than an exact computation
- SnapATAC v1 (r3fang) is superseded by SnapATAC2 (scverse/kaizhang), which scales to more than 10 million cells
- Iterative clustering/aggregation/peak-calling strategies of alternative workflows hinder application to large-scale datasets and underrepresent peaks in rare populations

## Evidence

- [results] SnapATAC's running time for dimensionality reduction scales linearly with cell number: [evidence span withheld — anchor status: not_found]
- [abstract] Nyström scalability claim: "Using the Nyström method, SnapATAC can process data from up to a million cells."
- [methods] runNormJaccard signature: "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [readme] Version constraints: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [readme] SnapATAC2 scalability alternative: [evidence span withheld — anchor status: external_doc]
