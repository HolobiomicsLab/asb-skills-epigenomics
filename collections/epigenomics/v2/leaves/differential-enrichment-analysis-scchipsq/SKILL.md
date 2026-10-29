---
name: differential-enrichment-analysis-scchipsq
description: Use when scChIP-seq count matrices have been clustered into epigenomic
  subpopulations (e.g. via ConsensusClusterPlus in ChromSCape) and you need per-cluster
  differentially enriched regions and biological interpretation.
when_to_use_negative:
- Input is raw unfiltered count matrices — cells must first pass QC (min 1600 unique
  reads, top 5% coverage removal) and correlation-based filtering
- Data are bulk ChIP-seq with high coverage — ChromSCape is designed for sparse single-cell
  histone modification data
- Only one cluster/subpopulation exists — differential enrichment requires at least
  two subpopulations to compare
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3173
tools:
- name: ChromSCape
  role: Interactive Shiny/R application that performs differential enrichment analysis
    between subpopulations and links regions to genes/pathways
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Runtime environment for ChromSCape analysis
- name: Bioconductor
  role: Distribution channel for the ChromSCape package
- name: Shiny
  role: User interface framework for interactive analysis in ChromSCape
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:d763c0f8bc088d3c4b15f4cfc87668028dd1289bfe1f4d72630f36abce9d796a
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# differential-enrichment-analysis-scchipsq

## Summary

Run per-cluster differential enrichment analysis in ChromSCape on H3K27me3 single-cell ChIP-seq data to identify genomic regions differentially enriched between epigenomic subpopulations, then interpret them via gene-set and pathway enrichment (MSigDB v5) using the Gencode_hg38_v26 transcriptome annotation.

## When to use

Use when scChIP-seq count matrices have been clustered into epigenomic subpopulations (e.g. via ConsensusClusterPlus in ChromSCape) and you need per-cluster differentially enriched regions and biological interpretation.

## When NOT to use

- Input is raw unfiltered count matrices — cells must first pass QC (min 1600 unique reads, top 5% coverage removal) and correlation-based filtering
- Data are bulk ChIP-seq with high coverage — ChromSCape is designed for sparse single-cell histone modification data
- Only one cluster/subpopulation exists — differential enrichment requires at least two subpopulations to compare

## Inputs

- Filtered scChIP-seq count matrix (regions detected in at least 1% of cells) for n=903 cells after QC and correlation filtering
- Cluster assignments from ConsensusClusterPlus at k=4 on the first 50 PCs
- Genomic regions (bins or peaks) with normalized coverage per cell
- Gencode_hg38_v26 reference transcriptome
- MSigDB v5 gene set database

## Outputs

- differential_enriched_regions_per_cluster.csv
- cluster_assignments.csv
- consensus_scores.csv
- Gene set / pathway enrichment results per cluster

## How to apply

In ChromSCape, after QC filtering (minimum 1600 unique reads per cell, top 5% coverage removal), correlation-based filtering, and PCA keeping the first 50 PCs with ConsensusClusterPlus clustering, run the differential enrichment module on the resulting subpopulations. For the TME application (HBCx-22, HBCx-22-TamR, HBCx-95, HBCx-95-CapaR H3K27me3 data, n=903 cells at k=4), tally differentially enriched regions per cluster and check against reported counts of 189/210/83/9 for C1-C4, verifying that cluster C3 is composed specifically of HBCx-95 (TNBC) cells. Link differentially enriched regions to genes using the Gencode_hg38_v26 human transcriptome reference and compute enrichments of MSigDB v5 gene sets within those regions to interpret each epigenome. Write cluster_assignments.csv, consensus_scores.csv, and differential_enriched_regions_per_cluster.csv as outputs.

## Related tools

- **ChromSCape** (Interactive Shiny/R application that performs differential enrichment analysis between subpopulations and links regions to genes/pathways) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Runtime environment for ChromSCape analysis)
- **Bioconductor** (Distribution channel for the ChromSCape package)
- **Shiny** (User interface framework for interactive analysis in ChromSCape)

## Examples

```
ChromSCape::differential_enriched_regions(scExp, cluster='ConsensusClusterPlus_k4', ref='Gencode_hg38_v26', enrichment='MSigDB_v5')
```

## Evaluation signals

- Differentially enriched region counts per cluster should be ~189/210/83/9 for C1–C4 in the TME k=4 analysis
- Cluster C3 should be almost exclusively composed of HBCx-95 (TNBC) cells
- Mean consensus score for selected clusters should be ~0.91 versus ~0.17 for others
- All output CSV files (cluster_assignments.csv, consensus_scores.csv, differential_enriched_regions_per_cluster.csv) should be present and well-formed
- Enriched gene sets should be reproducible from the MSigDB v5 database when linking regions to genes via Gencode_hg38_v26

## Limitations

- Benchmark comparison restricted to a subsampled set of 1287 cells and random sub-sampling of 500 MDA-MB-468 cells, a potentially limiting comparison scope
- Peak calling module requires user-supplied BAM files, which are not deposited for all benchmarking datasets
- No versioned release/archive (e.g. Zenodo DOI) provided for ChromSCape code; only hosted as a Bioconductor package and GitHub resources
- MACS2 is required for the peak calling module but its version is not specified

## Evidence

- [intro] Differentially enriched regions between subpopulations: "find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] MSigDB v5 gene set enrichment: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] Region filtering 1% of cells: "keeping regions detected in at least 1% of cells"
- [discussion] Minimum coverage 1600 reads: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] TME data source figshare: "matrices available at https://figshare.com/projects/Single-Cell_ChIP-seq_of_Mouse_Stromal_Cells_in_PDX_tumour_models_of_resistance/66419"
