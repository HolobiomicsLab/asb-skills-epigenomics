---
name: scientific-task-formulation
description: 'Use when a published finding must be turned into a reproducible task:
  the goal is to re-derive a specific result (e.g. replicate mixing after FastMNN
  correction without merging resistant-tumor cells) and the source text must be decomposed
  into datasets, tools, parameters and pass/fail criteria.'
when_to_use_negative:
- The input is already a feature table or batch-corrected embedding — formulation
  is done; move to execution/evaluation
- The goal is cross-method benchmarking (SnapATAC, CisTopic, Cusanovich2018, EpiScanpy
  vs ChromSCape) rather than reproducing one published finding — use the ChromSCape_benchmarking
  scripts instead
- Data are bulk ChIP-seq or outside the sparse single-cell histone-modification scope
  the article targets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application whose fastMNN batch-correction
    module is the subject of the formulated task; it implements counting, QC filtering,
    PCA-50, t-SNE/UMAP and FastMNN correction
- name: scChIPseq_DataEngineering pipeline
  role: Upstream data-engineering pipeline (schip_processing) that produces the exploitable
    count matrix from raw paired-end FASTQ via barcode mapping, trimming, deduplication
    and counting
  repo: https://github.com/vallotlab/scChIPseq_DataEngineering
- name: ChromSCape_benchmarking scripts
  role: Provide reproducible inputs and options (subsample - 500 cells, 50kbp-bin
    count matrices, BED files such as hg38-blacklist.v2.bed and merged_peaks.5000.bed)
    used to reproduce article figures
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:3221a164245db4d0f52f201ebd43be2cad64ac3675b6dc5c2186f11153381f63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/scChIPseq_DataEngineering
  tools:
  - ChromSCape
  - ChromSCape_benchmarking scripts
  - scChIPseq_DataEngineering pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scientific_task_formulation

## Summary

Converts an article-level finding (here, ChromSCape's fastMNN batch correction of HBCx-95 H3K27me3 scChIP-seq biological replicates) into an executable, evaluable task specification by pinning down the concrete workflow steps, tool versions, datasets, thresholds, and success criteria from the source text and repository documentation.

## When to use

Use when a published finding must be turned into a reproducible task: the goal is to re-derive a specific result (e.g. replicate mixing after FastMNN correction without merging resistant-tumor cells) and the source text must be decomposed into datasets, tools, parameters and pass/fail criteria.

## When NOT to use

- The input is already a feature table or batch-corrected embedding — formulation is done; move to execution/evaluation
- The goal is cross-method benchmarking (SnapATAC, CisTopic, Cusanovich2018, EpiScanpy vs ChromSCape) rather than reproducing one published finding — use the ChromSCape_benchmarking scripts instead
- Data are bulk ChIP-seq or outside the sparse single-cell histone-modification scope the article targets

## Inputs

- Gzipped single-cell BED files or precomputed H3K27me3 scChIP-seq count matrices in 50kbp bins for untreated HBCx-95 biological replicates and the resistant tumor (figshare: Single-Cell ChIP-seq of Mouse Stromal Cells in PDX tumour models of resistance)
- GEO-hosted datasets for related samples (GSE117309, GSE152502, GSE105012, GSE124680, GSE124690, GSE99172)
- Article section text containing the finding, workflow steps, and filter steps to be formalized
- Repository README (scChIPseq_DataEngineering, ChromSCape_benchmarking) describing pipeline stages and reproducibility options

## Outputs

- A written task specification: research question, ordered workflow steps (counting, QC filtering, region filtering, normalization, PCA-50, FastMNN correction, 2D visualization), tool list, and datasets
- Defined success criteria: replicate cells mix post-correction; resistant-tumor cells remain separated
- Explicit list of missing/unverifiable parameters (e.g. unspecified MACS2 version, subsample-500-cells scope)

## How to apply

Decompose the finding into the article's own pipeline: generate count matrices in 50kbp bins (or input them directly), apply QC filtering (minimum 1600 unique reads per cell, removal of the top 5% coverage cells), filter regions detected in at least 1% of cells, normalize by total read count and region size, run PCA keeping the first 50 PCs, and apply the FastMNN batch-correction option from the batchelor package inside ChromSCape. Bind the task to concrete datasets: the HBCx-95 untreated biological-replicate and resistant-tumor H3K27me3 scChIP-seq matrices on figshare, plus GEO accessions (e.g. GSE117309) where relevant. State the evaluation criterion verbatim from the finding: after correction, cells from the two untreated HBCx-95 replicates should mix in the t-SNE/UMAP embedding while remaining separated from resistant-tumor cells. When the section text lacks methods for a sub-step, mark it explicitly (as the source card does: 'No workflow can be specified') and reconstruct parameters from the Methods/discussion rather than inventing them.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application whose fastMNN batch-correction module is the subject of the formulated task; it implements counting, QC filtering, PCA-50, t-SNE/UMAP and FastMNN correction)
- **scChIPseq_DataEngineering pipeline** (Upstream data-engineering pipeline (schip_processing) that produces the exploitable count matrix from raw paired-end FASTQ via barcode mapping, trimming, deduplication and counting) — https://github.com/vallotlab/scChIPseq_DataEngineering
- **ChromSCape_benchmarking scripts** (Provide reproducible inputs and options (subsample - 500 cells, 50kbp-bin count matrices, BED files such as hg38-blacklist.v2.bed and merged_peaks.5000.bed) used to reproduce article figures) — https://github.com/vallotlab/ChromSCape_benchmarking

## Examples

```
schip_processing All -f FORWARD -r REVERSE -o OUTPUT -c CONFIG [-d] [-h] [-v]
```

## Evaluation signals

- After FastMNN correction, cells from the two untreated HBCx-95 biological replicates intermix in the t-SNE/UMAP embedding
- Resistant-tumor cells remain separated from untreated HBCx-95 cells post-correction, demonstrating no overcorrection of true biological differences
- QC invariant: retained cells have at least 1600 unique reads and cells in the top 5% coverage are removed; regions kept are detected in at least 1% of cells
- Dimensionality invariant: the PCA step retains exactly the first 50 centered PCs before FastMNN and 2D visualization
- Scope check: the specification explicitly flags any sub-step lacking methods in the source section rather than filling gaps with invented parameters

## Limitations

- The source section text for this sub-task provides no methods, parameters, or artifacts ('No workflow can be specified'), so the workflow must be reconstructed from the Methods/discussion and repositories
- Benchmark comparisons were restricted to a subsampled set of 1287 cells (and random sub-sampling of 500 MDA-MB-468 cells), limiting reproducibility scope
- Reproducibility gaps: MACS2 version unspecified, most R dependency versions unreported, no versioned archive (e.g. Zenodo DOI) for ChromSCape code, and no changelog in the supplementary information

## Evidence

- [discussion] A batch correction option using mutual nearest neighbors “FastMNN” function from “batchelor” package16 is implemented: "A batch correction option using mutual nearest neighbors “FastMNN” function from “batchelor” package16 is implemented"
- [discussion] a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] keeping regions detected in at least 1% of cells: "keeping regions detected in at least 1% of cells"
- [readme] Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’ to reproduce the analysis.: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’ to reproduce the analysis."
