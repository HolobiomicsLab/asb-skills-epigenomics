---
name: region-to-gene-annotation-mapping
description: Use when a cluster-vs-rest differential test has returned significant
  genomic regions (adj. p-value < 0.01, |fold change| > 1) and you need to convert
  those regions into gene lists for pathway over-representation analysis.
when_to_use_negative:
- Input is already a gene-level list or gene-expression matrix with no genomic region
  coordinates to map
- Data are not human and no Gencode_hg38_v26-equivalent transcriptome annotation is
  available
- The goal is motif/k-mer-level interpretation of scATAC-seq peaks rather than assigning
  differential regions to genes and gene sets
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application that finds differentially enriched
    regions between subpopulations, links regions to associated genes, and tests MSigDB
    v5 gene set enrichment
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: Bioconductor
  role: Distribution channel for ChromSCape, which implements the region-to-gene linkage
    using Gencode_hg38_v26
- name: Shiny
  role: Interactive interface through which the user runs the differential-enrichment
    and pathway-interpretation steps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:813d46c64ead397ac99c061ef24ade9f7e8dbe550ab1ee31b1d5c30efd7d3951
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# region-to-gene-annotation-mapping

## Summary

Maps significant differentially enriched chromatin regions (e.g., cluster-specific H3K27me3 loci) to associated genes using the Gencode_hg38_v26 human transcriptome reference, enabling downstream gene set enrichment against MSigDB v5. This is the interpretive bridge in ChromSCape between per-cluster differential testing and pathway-level biological interpretation.

## When to use

Use when a cluster-vs-rest differential test has returned significant genomic regions (adj. p-value < 0.01, |fold change| > 1) and you need to convert those regions into gene lists for pathway over-representation analysis.

## When NOT to use

- Input is already a gene-level list or gene-expression matrix with no genomic region coordinates to map
- Data are not human and no Gencode_hg38_v26-equivalent transcriptome annotation is available
- The goal is motif/k-mer-level interpretation of scATAC-seq peaks rather than assigning differential regions to genes and gene sets

## Inputs

- Filtered, normalized scChIP-seq count matrix (normalized by total read count and region size)
- Per-cell cluster labels from the TME H3K27me3 clustering
- Differential enrichment results per cluster (region, log-rank statistic/effect, p-value, BH-adjusted p-value, cluster)
- Reference human transcriptome annotation: Gencode_hg38_v26
- Gene set database: MSigDB v5

## Outputs

- differential_enrichment_results.tsv (region, log-rank statistic or effect, p-value, BH-adjusted p-value, cluster)
- gene_set_enrichment.tsv (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster)

## How to apply

Start from the filtered, normalized count matrix (normalized by total read count and region size) and the per-cell cluster labels from the tumor micro-environment H3K27me3 analysis. For each cluster, run a Wilcoxon rank-sum test of each region against all other cells and adjust p-values across regions with Benjamini-Hochberg; call regions significant when the BH-adjusted p-value < 0.01 and the absolute fold change > 1, classifying them as enriched or depleted for H3K27me3. Assign the significant loci to genes using the reference human transcriptome Gencode_hg38_v26. Then run a hypergeometric over-representation test of the genes linked to each cluster's differentially enriched regions against MSigDB v5 gene sets, adjusting enrichment p-values for multiple testing. Write per-cluster differential region statistics (region, effect/statistic, p-value, BH-adjusted p-value, cluster) to differential_enrichment_results.tsv and per-cluster enrichment statistics (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster) to gene_set_enrichment.tsv.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application that finds differentially enriched regions between subpopulations, links regions to associated genes, and tests MSigDB v5 gene set enrichment) — https://github.com/vallotlab/ChromSCape_benchmarking
- **Bioconductor** (Distribution channel for ChromSCape, which implements the region-to-gene linkage using Gencode_hg38_v26)
- **Shiny** (Interactive interface through which the user runs the differential-enrichment and pathway-interpretation steps)

## Evaluation signals

- differential_enrichment_results.tsv contains one entry block per cluster and includes a BH-adjusted p-value column
- Significant regions satisfy the ChromSCape cutoffs: BH-adjusted p-value < 0.01 and absolute fold change > 1, classified as enriched or depleted for H3K27me3
- gene_set_enrichment.tsv lists MSigDB v5 gene sets with hypergeometric p-values for each tested cluster
- Each enrichment entry reports an overlap size (number of linked genes in the gene set), not just a p-value
- Both TSV files exist and every cluster tested has rows in both outputs

## Limitations

- Region-to-gene linkage relies on the reference human transcriptome Gencode_hg38_v26, so the workflow is human-specific as described
- CNV-containing regions should be excluded via a user-supplied BED (e.g., 77 Mbp of identified MDA-MB-468 copy-number aberrant regions), otherwise differential regions and downstream gene lists can be confounded
- Sparse single-cell histone modification data (low coverage per cell) limits the power of the differential test feeding the gene lists
- No versioned release or Zenodo DOI is provided for ChromSCape code; only the Bioconductor package and GitHub resources, limiting exact environment reproducibility

## Evidence

- [discussion] differentially enriched regions linked to genes for gene set enrichment: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] Gencode_hg38_v26 is the region-to-gene reference: "ChromSCape uses the reference human transcriptome Gencode_hg38_v26"
- [intro] Interpret epigenomes by linking regions to genes and pathways: "The user can interactively identify subpopulations with common epigenomes within heterogeneous samples, find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] Normalized count matrix input for downstream differential and gene-linkage steps: "Post quality control filtering, the matrices are normalized by total read count and region size"
