---
name: wilcoxon-rank-sum-testing-on-sparse-count-matrices
description: Use when you have a filtered, normalized single-cell chromatin count
  matrix (scChIP-seq/scATAC-seq, e.g. 50kbp bins or peaks) plus per-cell cluster labels,
  and you need cluster-specific enriched/depleted regions for downstream gene set
  interpretation.
when_to_use_negative:
- The count matrix is not yet quality-control filtered and normalized by total read
  count and region size — test on unnormalized sparse counts will be confounded by
  cell coverage
- There are no per-cell cluster labels; the test is a cluster-vs-rest design and requires
  prior subpopulation identification
- The goal is motif- or k-mer-level feature analysis rather than per-region differential
  enrichment between defined subpopulations
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0182
tools:
- name: ChromSCape
  role: Implements the cluster-vs-rest Wilcoxon rank-sum differential test on normalized
    single-cell count matrices and links significant regions to genes for MSigDB enrichment
- name: Shiny
  role: Interactive interface through which the user runs the differential-enrichment
    module on identified subpopulations
- name: R
  role: Language in which the differential-enrichment and gene set enrichment statistics
    are computed
- name: Bioconductor
  role: Distribution channel of ChromSCape as a package providing the differential
    analysis functions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:c94e74232b37ac9d7fcafbeb53ad9e6765935efeb67ad1b9adaa54736333e4fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0182
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Wilcoxon rank-sum testing on sparse count matrices

## Summary

A differential-enrichment strategy from ChromSCape that tests each genomic region with a per-cluster versus-all-other-cells Wilcoxon rank-sum test on normalized single-cell counts, then calls regions enriched or depleted when the BH-adjusted p-value is <0.01 and the absolute fold change exceeds 1.

## When to use

Use when you have a filtered, normalized single-cell chromatin count matrix (scChIP-seq/scATAC-seq, e.g. 50kbp bins or peaks) plus per-cell cluster labels, and you need cluster-specific enriched/depleted regions for downstream gene set interpretation.

## When NOT to use

- The count matrix is not yet quality-control filtered and normalized by total read count and region size — test on unnormalized sparse counts will be confounded by cell coverage
- There are no per-cell cluster labels; the test is a cluster-vs-rest design and requires prior subpopulation identification
- The goal is motif- or k-mer-level feature analysis rather than per-region differential enrichment between defined subpopulations

## Inputs

- Filtered, normalized single-cell count matrix (normalized by total read count and region size)
- Per-cell cluster labels from the upstream clustering step (e.g. TME H3K27me3 subpopulations)
- Gencode_hg38_v26 human transcriptome reference for region-to-gene assignment
- MSigDB v5 gene set database

## Outputs

- differential_enrichment_results.tsv — per-cluster region statistics (region, log-rank statistic or effect, p-value, BH-adjusted p-value, cluster)
- gene_set_enrichment.tsv — per-cluster gene set statistics (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster)
- Lists of regions classified as enriched or depleted per cluster

## How to apply

Load the count matrix normalized by total read count and region size together with cluster labels from the tumor micro-environment H3K27me3 analysis. For every cluster, apply a Wilcoxon rank-sum test on the normalized counts of each region, comparing that cluster against all other cells (cluster-vs-rest design), then adjust p-values across regions with the Benjamini-Hochberg procedure. Call a region differentially enriched or depleted when the adjusted p-value is below 0.01 and the absolute fold change is greater than 1. Assign significant regions to genes using the Gencode_hg38_v26 human transcriptome reference, then test the resulting gene lists against MSigDB v5 gene sets with a hypergeometric over-representation test, adjusting enrichment p-values for multiple testing. Rationale: bulk ChIP-seq tools are not fit for the low coverage and sparsity of single-cell histone modification datasets, so a rank-based test on normalized per-cell counts avoids parametric assumptions about sparse data.

## Related tools

- **ChromSCape** (Implements the cluster-vs-rest Wilcoxon rank-sum differential test on normalized single-cell count matrices and links significant regions to genes for MSigDB enrichment)
- **Shiny** (Interactive interface through which the user runs the differential-enrichment module on identified subpopulations)
- **R** (Language in which the differential-enrichment and gene set enrichment statistics are computed)
- **Bioconductor** (Distribution channel of ChromSCape as a package providing the differential analysis functions)

## Evaluation signals

- Both TSV outputs exist: differential_enrichment_results.tsv and gene_set_enrichment.tsv
- differential_enrichment_results.tsv contains one entry block per cluster and includes a BH-adjusted p-value column
- Significant regions satisfy both thresholds: BH-adjusted p-value < 0.01 AND absolute fold change > 1
- gene_set_enrichment.tsv lists MSigDB v5 gene sets with hypergeometric p-values (and adjusted p-values) for each tested cluster
- Every region tested has a p-value adjusted across regions by Benjamini-Hochberg, not left uncorrected

## Limitations

- Differential regions must be filtered out upstream (e.g. regions detected in at least 1% of cells, exclusion of CNV regions via user-provided BED) or CNV artefacts can drive apparent cluster-specific enrichment
- Region-to-gene assignment is limited to the Gencode_hg38_v26 human transcriptome reference, so the approach is human-specific as described
- Gene set interpretation depends on the MSigDB v5 database version; other versions may give non-comparable enrichment results

## Evidence

- [intro] The user can interactively identify subpopulations with common epigenomes within heterogeneous samples, find differentially enriched regions between subpopulations and interpret epigenomes by linking: "The user can interactively identify subpopulations with common epigenomes within heterogeneous samples, find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] Post quality control filtering, the matrices are normalized by total read count and region size: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] enriched gene sets of the MSigDB v5 database28 within differentially enriched regions: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] ChromSCape uses the reference human transcriptome Gencode_hg38_v26: "ChromSCape uses the reference human transcriptome Gencode_hg38_v26"
- [intro] Existing tools used to analyze bulk ChIP-seq experiments are not fit for the low coverage and sparsity of these single-cell histone modifications datasets: "Existing tools used to analyze bulk ChIP-seq experiments are not fit for the low coverage and sparsity of these single-cell histone modifications datasets"
