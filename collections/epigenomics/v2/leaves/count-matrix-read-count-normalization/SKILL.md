---
name: count-matrix-read-count-normalization
description: Reference for converting a raw single-cell genomic bin/peak count matrix
  into a filtered, normalized matrix prior to PCA/clustering, when cell coverage varies
  and sparse outlier cells or regions must be removed.
when_to_use_negative:
- The input is already a normalized or batch-corrected feature/embedding matrix (e.g.
  50-PC PCA output or UMAP coordinates) rather than raw per-cell counts
- The data are bulk ChIP-seq read counts — the article states existing bulk tools/parameters
  are not fit for the low coverage and sparsity of single-cell histone modification
  datasets
- Peak calling on raw BAM files is needed first; that upstream module (requiring MACS2)
  must run before count-matrix QC and normalization
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application that generates the count matrix, applies
    the QC cell/region filters, and normalizes by total read count and region size
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which the filtering and normalization of the count matrix are
    implemented and downstream R analysis runs
- name: Bioconductor
  role: Distribution channel for the ChromSCape package performing the QC and normalization
    steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:fb46a7f26be8e91d3279bece8af988fd9ea0ee5f1563240ec40da3bfcad374ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# count-matrix-read-count-normalization

## Summary

Quality-control filtering and normalization of single-cell epigenomic count matrices (e.g. scChIP-seq/scATAC-seq binned counts) as implemented in ChromSCape: cells are filtered on read-count thresholds and coverage-percentile outliers, regions are filtered on detection frequency, and the surviving matrix is normalized by total read count and region size.

## When to use

Reference for converting a raw single-cell genomic bin/peak count matrix into a filtered, normalized matrix prior to PCA/clustering, when cell coverage varies and sparse outlier cells or regions must be removed.

## When NOT to use

- The input is already a normalized or batch-corrected feature/embedding matrix (e.g. 50-PC PCA output or UMAP coordinates) rather than raw per-cell counts
- The data are bulk ChIP-seq read counts — the article states existing bulk tools/parameters are not fit for the low coverage and sparsity of single-cell histone modification datasets
- Peak calling on raw BAM files is needed first; that upstream module (requiring MACS2) must run before count-matrix QC and normalization

## Inputs

- Single-cell count matrix from BAM/BED files aggregated into genomic bins, peaks, or TSS regions (rows = regions, columns = cells)
- Per-cell read coverage values used to set the minimum read count and upper-percentile thresholds
- Optionally, a user-provided BED file of regions to exclude (e.g. CNV regions such as MM468_identified_CNA.bed)

## Outputs

- Filtered count matrix retaining cells passing the 1600-read minimum and non-top-5% coverage filters
- Normalized count matrix (e.g. count_matrix_normalized.tsv) scaled by total read count and region size, ready for PCA/50-PC reduction

## How to apply

Aggregate per-cell signal into genomic bins (50,000 bp for broad marks like H3K27me3; 5,000 bp or TSS ±2500 bp windows using Gencode_hg38_v26 for sharp marks like H3K4me3), or start from a BAM/BED-derived count matrix. Filter cells first by a minimum coverage threshold (e.g. 1600 unique reads per cell), then remove the top-percentile coverage cells (default top 5%) to eliminate coverage outliers. Filter regions by keeping only those detected in at least 1% of cells and supported by a user-defined percentage of cells with coverage greater than 1000 reads. Post quality-control filtering, normalize the matrix by total read count and region size so that deep-sequenced cells and large bins do not dominate downstream dimensionality reduction; the resulting matrix (e.g. count_matrix_normalized.tsv) feeds directly into PCA (50 PCs), t-SNE/UMAP, and clustering.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application that generates the count matrix, applies the QC cell/region filters, and normalizes by total read count and region size) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which the filtering and normalization of the count matrix are implemented and downstream R analysis runs)
- **Bioconductor** (Distribution channel for the ChromSCape package performing the QC and normalization steps)

## Examples

```
schip_processing All -f FORWARD -r REVERSE -o OUTPUT -c CONFIG
```

## Evaluation signals

- Output matrix exists at the expected path (e.g. count_matrix_normalized.tsv) and is parseable with expected region rows and cell columns
- Column count matches cells passing the 1600-unique-read minimum and the top-5%-coverage removal; verify no retained cell falls below the minimum or sits above the upper percentile
- Row count matches regions detected in at least 1% of cells and supported by the user-defined percentage of cells with coverage >1000 reads
- Row sums of the normalized matrix are comparable across cells (total-read-count scaling removed library-size differences); bin values scale inversely with region size
- Downstream PCA on the normalized matrix yields 50 PCs and t-SNE/UMAP embeds cells into distinguishable subpopulations, matching the article's benchmark behavior on the H3K27me3 in-silico mix (Jurkat, Ramos, HBCx-22, MDA-MB-468)

## Limitations

- Thresholds are user-dependent: the minimum read count per cell and the upper percentile of removed cells are set interactively, so results are not fully parameter-free
- The region filter requires a user-defined percentage of cells with coverage greater than 1000 reads; sparse datasets may lose many regions under this rule
- Benchmark scope was restricted to a subsampled set of 1287 cells (and random sub-sampling of 500 MDA-MB-468 cells), so behavior on much larger datasets is not established by the article
- Pre-normalization deduplication (PCR, RT, window duplicate removal) is assumed done upstream in the data-engineering pipeline; applying QC/normalization to non-deduplicated counts can bias coverage estimates
- Most R dependency versions are unreported (only STAR v2.7.0 versioned), limiting reproducibility of the exact filtering environment

## Evidence

- [discussion] Post quality control filtering, the matrices are normalized by total read count and region size: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] Minimum coverage of 1600 unique reads per cell and top-5% removal: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Region filtering thresholds: "Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out"
- [discussion] Count matrix generation context: "ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites"
- [readme] Upstream deduplication feeding the count matrix: [evidence span withheld — anchor status: external_doc]
