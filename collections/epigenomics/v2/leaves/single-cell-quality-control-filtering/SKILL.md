---
name: single-cell-quality-control-filtering
description: Use when raw single-cell count matrices (50kbp/5kbp bins or TSS regions)
  from scChIP-seq/scCUT&Tag/scChIC-seq must be cleaned of low-coverage and outlier
  cells before dimensionality reduction.
when_to_use_negative:
- Input is already a QC-filtered count matrix from a prior ChromSCape run — reapplying
  coverage filters could remove valid cells
- Data is bulk ChIP-seq or scRNA-seq where per-cell read coverage sparsity thresholds
  (1600 unique reads) do not apply
- The analysis goal is upstream raw-read processing (FASTQ-to-count-matrix), which
  requires the scChIPseq_DataEngineering pipeline instead of this QC module
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application implementing the minimum read count threshold,
    top-percentile coverage removal, correlation-based cell filtering, and region-detection
    filters
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: scChIPseq_DataEngineering
  role: Upstream pipeline whose duplicate-removal and counting steps produce the count
    matrix subsequently QC-filtered in ChromSCape
  repo: https://github.com/vallotlab/scChIPseq_DataEngineering
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:cfb0ac32d3e6f93b055e73425524834a1213e26f7e1dbdb3b8c53b11e7e4bd35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - ConsensusClusterPlus
  - R
  - Shiny
  - scChIPseq_DataEngineering
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# single-cell-quality-control-filtering

## Summary

ChromSCape's cell- and region-level quality-control filtering for sparse single-cell histone modification data (scChIP-seq, scCUT&Tag, scChIC-seq): enforcing minimum per-cell read coverage, removing top-percentile coverage outliers, and filtering unsupported regions before normalization and PCA.

## When to use

Use when raw single-cell count matrices (50kbp/5kbp bins or TSS regions) from scChIP-seq/scCUT&Tag/scChIC-seq must be cleaned of low-coverage and outlier cells before dimensionality reduction.

## When NOT to use

- Input is already a QC-filtered count matrix from a prior ChromSCape run — reapplying coverage filters could remove valid cells
- Data is bulk ChIP-seq or scRNA-seq where per-cell read coverage sparsity thresholds (1600 unique reads) do not apply
- The analysis goal is upstream raw-read processing (FASTQ-to-count-matrix), which requires the scChIPseq_DataEngineering pipeline instead of this QC module

## Inputs

- Aggregated single-cell count matrix (gzipped BED/BAM files converted into genomic bins or TSS ±2500bp regions)
- Per-cell coverage/read-count metadata (unique reads per cell)
- Optional user-provided BED region list to exclude (e.g. CNV regions)

## Outputs

- Filtered count matrix retaining cells passing the 1600-read minimum and top-5%-coverage removal
- Filtered region set (regions detected in ≥1% of cells and supported by cells with >1000 reads)
- Normalized count matrix (total read count and region size scaling), e.g. count_matrix_normalized.tsv

## How to apply

Load the aggregated single-cell count matrix into ChromSCape (Shiny/R Bioconductor app) and set a threshold on a minimum read count per cell — the paper uses 1600 unique reads per cell — plus an upper percentile to remove outlier cells (default: top 5% coverage). Optionally apply correlation-based filtering, removing cells whose Pearson's pairwise correlation score falls below a threshold t with at least p% of cells (p = 1% by default). Then filter regions: keep only regions detected in at least 1% of cells, and drop regions not supported by a user-defined percentage of cells with coverage greater than 1000 reads. After filtering, normalize by total read count and region size before PCA with centering keeping the first 50 PCs. The rationale is that single-cell histone modification data is sparse and low-coverage, so bulk ChIP-seq tools and unfiltered matrices are unfit for downstream clustering.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application implementing the minimum read count threshold, top-percentile coverage removal, correlation-based cell filtering, and region-detection filters) — https://github.com/vallotlab/ChromSCape_benchmarking
- **scChIPseq_DataEngineering** (Upstream pipeline whose duplicate-removal and counting steps produce the count matrix subsequently QC-filtered in ChromSCape) — https://github.com/vallotlab/scChIPseq_DataEngineering

## Examples

```
schip_processing All -f FORWARD -r REVERSE -o OUTPUT -c CONFIG
```

## Evaluation signals

- Retained cells have at least 1600 unique reads and cells in the top 5% coverage percentile are removed
- Every retained region is detected in at least 1% of cells after filtering
- Retained cells pass the correlation-based filter (Pearson pairwise correlation above threshold t with ≥1% of cells)
- Downstream output matrix row/column counts match expected retention under the 1600-read and top-5% filters, and normalized values reflect total-read-count and region-size scaling
- Filtered matrices support successful downstream clustering (e.g. ARI = 0.976 K562/H1 separation, k=4 consensus clustering of 903 TME cells)

## Limitations

- Thresholds (1600 unique reads, top 5% removal, 1% region detection) are dataset-calibrated defaults and may need tuning for very low-coverage or small-n datasets
- Correlation-based filtering parameters (t, p=1%) and region support thresholds are user-defined, so results depend on user choices not fixed by the software
- Correlation filtering over aggressive settings could discard biologically distinct populations; benchmark scope was limited to a subsampled set of 1287 cells

## Evidence

- [discussion] Minimum read count per cell threshold: "the user sets a threshold on a minimum read count per cell and the upper percentile of cells to remove"
- [discussion] 1600 reads / top 5% coverage cutoff: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Correlation-based cell filtering: "cells displaying a Pearson’s pairwise correlation score below a threshold t with at least p% of cells are filtered out (p is set at 1% by default)"
- [discussion] Region support filter: "Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out"
- [readme] Upstream duplicate removal in data-engineering pipeline: [evidence span withheld — anchor status: external_doc]
