---
name: hypergeometric-gene-set-overrepresentation-analysis
description: Use when you have per-cluster differentially enriched regions from single-cell
  ChIP/epigenomic data (e.g. H3K27me3 scChIP-seq) and need pathway-level interpretation
  by gene-set over-representation.
when_to_use_negative:
- Input is single-cell transcriptomic data where standard GSEA-style ranking of expressed
  genes applies — this workflow is built for region-level chromatin data that must
  first be linked to genes via Gencode_hg38_v26.
- No cluster-level differential regions available (e.g. only a count matrix without
  cluster labels or differential test results) — there is no gene list to over-representation-test.
- Non-human epigenomic data — the region-to-gene linking in ChromSCape uses the human
  Gencode_hg38_v26 reference.
edam_operation: http://edamontology.org/operation_3759
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application that performs the per-cluster-vs-rest
    Wilcoxon differential test, links significant regions to genes, and runs MSigDB
    v5 hypergeometric enrichment
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: Shiny
  role: User interface through which the differential-enrichment and gene-set enrichment
    modules are driven
- name: R
  role: Implementation language for the differential testing and hypergeometric enrichment
    computations
- name: Bioconductor
  role: Distribution channel for the ChromSCape package executing this workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:c3b55193b8f5a71883fe6e9a9546d5769f84a3361721bee13afbf69ac4cd6905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3759
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hypergeometric-gene-set-overrepresentation-analysis

## Summary

Links cluster-specific differentially enriched (or depleted) chromatin regions to genes and runs a hypergeometric over-representation test against MSigDB v5 gene sets, with multiple-testing correction, to interpret cluster epigenomes in ChromSCape.

## When to use

Use when you have per-cluster differentially enriched regions from single-cell ChIP/epigenomic data (e.g. H3K27me3 scChIP-seq) and need pathway-level interpretation by gene-set over-representation.

## When NOT to use

- Input is single-cell transcriptomic data where standard GSEA-style ranking of expressed genes applies — this workflow is built for region-level chromatin data that must first be linked to genes via Gencode_hg38_v26.
- No cluster-level differential regions available (e.g. only a count matrix without cluster labels or differential test results) — there is no gene list to over-representation-test.
- Non-human epigenomic data — the region-to-gene linking in ChromSCape uses the human Gencode_hg38_v26 reference.

## Inputs

- Filtered, normalized single-cell count matrix (normalized by total read count and region size)
- Per-cell cluster labels from the TME H3K27me3 scChIP-seq analysis
- Differential enrichment results per cluster (BH-adjusted p-values, fold change, enriched/depleted classification)
- Region-to-gene assignment reference: Gencode_hg38_v26 human transcriptome
- Gene set database: MSigDB v5

## Outputs

- differential_enrichment_results.tsv — per-cluster differential region statistics (region, log-rank statistic or effect, p-value, BH-adjusted p-value, cluster)
- gene_set_enrichment.tsv — per-cluster gene set enrichment statistics (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster)

## How to apply

Start from the filtered, normalized count matrix (normalized by total read count and region size) plus per-cell cluster labels, and run a per-cluster-vs-rest Wilcoxon rank-sum test on each region, Benjamini-Hochberg-adjusting p-values across regions. Classify regions as significantly enriched or depleted using an adjusted p-value cutoff of 0.01 and an absolute fold-change cutoff greater than 1, as ChromSCape does in the tumor micro-environment H3K27me3 analysis. Assign significant regions to genes using the Gencode_hg38_v26 human transcriptome reference. For each cluster, test the linked gene list against MSigDB v5 gene sets with a hypergeometric over-representation test and adjust the enrichment p-values for multiple testing. Rationale: sparsity of scChIP-seq data means per-region signals are weak individually, so aggregating region-to-gene links into gene-set tests gives interpretable cluster-specific biology.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application that performs the per-cluster-vs-rest Wilcoxon differential test, links significant regions to genes, and runs MSigDB v5 hypergeometric enrichment) — https://github.com/vallotlab/ChromSCape_benchmarking
- **Shiny** (User interface through which the differential-enrichment and gene-set enrichment modules are driven)
- **R** (Implementation language for the differential testing and hypergeometric enrichment computations)
- **Bioconductor** (Distribution channel for the ChromSCape package executing this workflow)

## Evaluation signals

- Both output files exist: differential_enrichment_results.tsv and gene_set_enrichment.tsv.
- differential_enrichment_results.tsv contains one entry block per cluster and includes a BH-adjusted p-value column.
- gene_set_enrichment.tsv lists MSigDB v5 gene sets with hypergeometric p-values for each tested cluster.
- All BH-adjusted p-values fall in [0,1] and overlap sizes do not exceed the gene-set size or the tested gene-list size.
- Regions classified as significant satisfy adjusted p-value < 0.01 and absolute fold change > 1 before gene linking.

## Limitations

- Enrichment quality depends on the region significance cutoffs (adjusted p < 0.01, |fold change| > 1); overly stringent or permissive thresholds change the gene lists entering the hypergeometric test.
- Gene linking is restricted to the Gencode_hg38_v26 human transcriptome reference, so non-human or poorly annotated regions are dropped.
- The MSigDB v5 database version used is dated; newer gene set releases are not reflected in the published analysis.
- Sparse single-cell histone-modification data yield few significant regions per cluster, which can make hypergeometric overlap statistics unstable.

## Evidence

- [intro] Linking regions to genes and MSigDB enrichment: "The user can interactively identify subpopulations with common epigenomes within heterogeneous samples, find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] MSigDB v5 gene sets tested within differentially enriched regions: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] Gencode_hg38_v26 as region-to-gene reference: "ChromSCape uses the reference human transcriptome Gencode_hg38_v26"
- [discussion] Normalization of count matrices prior to downstream testing: "Post quality control filtering, the matrices are normalized by total read count and region size"
