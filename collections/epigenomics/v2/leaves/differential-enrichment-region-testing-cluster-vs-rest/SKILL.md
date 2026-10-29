---
name: differential-enrichment-region-testing-cluster-vs-rest
description: Use when a filtered, normalized single-cell epigenomic count matrix (H3K27me3
  scChIP-seq or scATAC-seq) has per-cell cluster labels and you must find regions
  enriched or depleted per cluster versus all other cells, then interpret them via
  gene set enrichment.
when_to_use_negative:
- The input is bulk ChIP-seq data — ChromSCape is built for the low coverage and sparsity
  of single-cell histone modification datasets
- Clusters have not been defined yet (no per-cell labels) — run QC filtering, PCA
  (50 PCs), t-SNE/UMAP and clustering first
- The goal is only visualizing cell populations without needing cluster-specific chromatin
  features or pathway interpretation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application that runs the per-cluster differential enrichment
    testing and links regions to genes and pathways
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: Shiny
  role: Interactive interface through which the differential enrichment module is
    operated
- name: R
  role: Implementation language for the Wilcoxon rank-sum testing, BH adjustment,
    and hypergeometric enrichment
- name: Bioconductor
  role: Distribution channel of the ChromSCape package used for this analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:de3a50fabda1b4b914e791775dc7cb2bbcfb3cad4030b046830fbe92c3ac4b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Differential-enrichment region testing (cluster-vs-rest) with MSigDB pathway enrichment

## Summary

Per-cluster differential testing of genomic regions in single-cell epigenomic count matrices (e.g., H3K27me3 scChIP-seq from the tumor micro-environment), followed by linking significant regions to genes and running MSigDB v5 gene set enrichment. It turns ChromSCape's epigenomic clusters into interpretable, cluster-specific chromatin features.

## When to use

Use when a filtered, normalized single-cell epigenomic count matrix (H3K27me3 scChIP-seq or scATAC-seq) has per-cell cluster labels and you must find regions enriched or depleted per cluster versus all other cells, then interpret them via gene set enrichment.

## When NOT to use

- The input is bulk ChIP-seq data — ChromSCape is built for the low coverage and sparsity of single-cell histone modification datasets
- Clusters have not been defined yet (no per-cell labels) — run QC filtering, PCA (50 PCs), t-SNE/UMAP and clustering first
- The goal is only visualizing cell populations without needing cluster-specific chromatin features or pathway interpretation

## Inputs

- Filtered, normalized single-cell epigenomic count matrix (normalized by total read count and region size; 50kbp bins or peaks)
- Per-cell cluster labels from the ChromSCape clustering (e.g., TME H3K27me3 clusters)
- Gencode_hg38_v26 human transcriptome reference for region-to-gene assignment
- MSigDB v5 gene set database

## Outputs

- differential_enrichment_results.tsv (region, log-rank statistic or effect, p-value, BH-adjusted p-value, cluster; one entry block per cluster)
- gene_set_enrichment.tsv (MSigDB v5 gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster)

## How to apply

Start from the filtered, normalized count matrix (normalized by total read count and region size, e.g., 50kbp bins) with per-cell cluster labels from the TME H3K27me3 analysis. For every cluster, run a Wilcoxon rank-sum test on normalized counts of each region comparing that cluster against all other cells (cluster-vs-rest design), then adjust p-values across regions with the Benjamini-Hochberg procedure. Call regions significant when the adjusted p-value is lower than 0.01 and the absolute fold change is greater than 1, classifying them as enriched or depleted for H3K27me3. Assign significant regions to genes using the Gencode_hg38_v26 human transcriptome reference, then run a hypergeometric over-representation test of MSigDB v5 gene sets on the linked genes per cluster, adjusting enrichment p-values for multiple testing. Write per-cluster region statistics (region, effect/statistic, p-value, BH-adjusted p-value, cluster) to differential_enrichment_results.tsv and per-cluster gene set statistics (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster) to gene_set_enrichment.tsv.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application that runs the per-cluster differential enrichment testing and links regions to genes and pathways) — https://github.com/vallotlab/ChromSCape_benchmarking
- **Shiny** (Interactive interface through which the differential enrichment module is operated)
- **R** (Implementation language for the Wilcoxon rank-sum testing, BH adjustment, and hypergeometric enrichment)
- **Bioconductor** (Distribution channel of the ChromSCape package used for this analysis)

## Evaluation signals

- Both output files exist: differential_enrichment_results.tsv and gene_set_enrichment.tsv
- differential_enrichment_results.tsv contains one entry block per cluster and includes a BH-adjusted p-value column
- gene_set_enrichment.tsv lists MSigDB v5 gene sets with hypergeometric p-values for each tested cluster
- Significant regions are classified as enriched or depleted only when adjusted p < 0.01 and absolute fold change > 1
- Enrichment p-values are multiple-testing adjusted (not raw hypergeometric p-values)

## Limitations

- Uses MSigDB v5 and Gencode_hg38_v26 as fixed references; newer gene set releases and transcriptome annotations are not covered
- Most R dependency versions are unreported in the article, limiting environment reproducibility of this module
- Benchmarking was restricted to a subsampled set of 1287 cells (500 MDA-MB-468 cells), so differential results on much larger or sparser datasets may behave differently

## Evidence

- [intro] find differentially enriched regions between subpopulations and interpret epigenomes by linking: "find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] Post quality control filtering, the matrices are normalized by total read count and region size: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] enriched gene sets of the MSigDB v5 database28 within differentially enriched regions: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] ChromSCape uses the reference human transcriptome Gencode_hg38_v26: "ChromSCape uses the reference human transcriptome Gencode_hg38_v26"
- [readme] Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’"
