---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a family of hypothesis tests has been run per cluster (cluster-vs-rest
  Wilcoxon tests across regions, or hypergeometric tests across MSigDB v5 gene sets)
  and raw p-values must be adjusted before thresholding significant regions or pathways.
when_to_use_negative:
- Only a single hypothesis test (or a tiny pre-specified set) is being run, so family-wise
  FDR control is unnecessary and would over-penalize
- The p-values are already adjusted (e.g. loaded from a prior differential_enrichment_results.tsv)
  — re-applying BH would double-correct and inflate p-values
- The goal is per-peak raw significance inspection or effect-size ranking rather than
  controlled selection of significant regions/gene sets
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application whose differential-enrichment module runs
    the per-cluster Wilcoxon tests and applies BH adjustment to region and gene-set
    p-values
- name: R
  role: Language in which the BH procedure (e.g. p.adjust with method='BH') is executed
    within the ChromSCape pipeline
- name: Bioconductor
  role: Distribution channel for the ChromSCape package implementing the corrected
    differential and enrichment testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:d25c9098b30baf71967d7ccc3a483a87595a096debf20a19288f1925ff4bf788
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to p-value families generated in the ChromSCape workflow — per-cluster Wilcoxon rank-sum tests over genomic regions and hypergeometric tests over MSigDB v5 gene sets — so that differentially enriched H3K27me3 regions and enriched pathways are called at controlled FDR.

## When to use

Use when a family of hypothesis tests has been run per cluster (cluster-vs-rest Wilcoxon tests across regions, or hypergeometric tests across MSigDB v5 gene sets) and raw p-values must be adjusted before thresholding significant regions or pathways.

## When NOT to use

- Only a single hypothesis test (or a tiny pre-specified set) is being run, so family-wise FDR control is unnecessary and would over-penalize
- The p-values are already adjusted (e.g. loaded from a prior differential_enrichment_results.tsv) — re-applying BH would double-correct and inflate p-values
- The goal is per-peak raw significance inspection or effect-size ranking rather than controlled selection of significant regions/gene sets

## Inputs

- Vector/table of raw Wilcoxon rank-sum p-values, one per genomic region per cluster (cluster-vs-rest design)
- Vector/table of raw hypergeometric over-representation p-values, one per MSigDB v5 gene set per cluster
- Normalized single-cell count matrix (H3K27me3 scChIP-seq, normalized by total read count and region size) with per-cell cluster labels

## Outputs

- BH-adjusted p-value column appended to per-cluster differential region statistics (region, effect, p-value, adjusted p-value, cluster) written to differential_enrichment_results.tsv
- Adjusted enrichment p-values in gene_set_enrichment.tsv (gene set, overlap size, hypergeometric p-value, adjusted p-value, cluster)
- Set of significant differentially enriched/depleted regions per cluster (BH-adjusted p < 0.01, |fold change| > 1) feeding MSigDB v5 pathway interpretation

## How to apply

In ChromSCape's differential-enrichment module, each cluster is tested against all other cells with a Wilcoxon rank-sum test on normalized counts (matrices normalized by total read count and region size); the resulting per-region p-values are adjusted across regions with the Benjamini-Hochberg procedure, and regions are called significant when the BH-adjusted p-value is below 0.01 with absolute fold change greater than 1, then classified as enriched or depleted for H3K27me3. A second BH correction is applied to the hypergeometric over-representation p-values of MSigDB v5 gene sets run on genes linked to each cluster's significant regions (regions assigned to genes via the Gencode_hg38_v26 reference). The rationale is that tens of thousands of genomic regions and thousands of gene sets are tested per cluster, so raw p-values would yield many false positives; BH controls the FDR across the whole family. Adjusted values, not raw ones, must be used for all significance calls and downstream pathway interpretation.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application whose differential-enrichment module runs the per-cluster Wilcoxon tests and applies BH adjustment to region and gene-set p-values)
- **R** (Language in which the BH procedure (e.g. p.adjust with method='BH') is executed within the ChromSCape pipeline)
- **Bioconductor** (Distribution channel for the ChromSCape package implementing the corrected differential and enrichment testing)

## Examples

```
p.adjust(raw_pvals, method = "BH")
```

## Evaluation signals

- differential_enrichment_results.tsv exists and contains one entry block per cluster with an explicit BH-adjusted p-value column
- gene_set_enrichment.tsv lists MSigDB v5 gene sets with hypergeometric p-values plus adjusted p-values for each tested cluster
- Invariant: every adjusted p-value is >= its corresponding raw p-value, and all adjusted p-values are within [0,1]
- Regions flagged as significant satisfy BH-adjusted p < 0.01 AND |fold change| > 1; depleted vs enriched classification is consistent with fold-change sign
- For any rank k, the k-th smallest adjusted p-value does not decrease when k increases (BH step-up monotonicity)

## Limitations

- BH control assumes reasonably independent or positively correlated tests; the article's region-level tests over correlated genomic bins/peaks may violate strict independence
- Correction is applied per cluster within a family, so adjusted p-values are not comparable across clusters with different numbers of tested regions
- Benchmark scope was limited (e.g. 1287 cells and 500 subsampled MDA-MB-468 cells), so the calibration of significance thresholds rests on that subsampled comparison

## Evidence

- [intro] find differentially enriched regions between subpopulations and interpret epigenomes by linking: "The user can interactively identify subpopulations with common epigenomes within heterogeneous samples, find differentially enriched regions between subpopulations and interpret epigenomes by linking"
- [discussion] enriched gene sets of the MSigDB v5 database within differentially enriched regions: "enriched gene sets of the MSigDB v5 database28 within differentially enriched regions"
- [discussion] the matrices are normalized by total read count and region size: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [readme] Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’ to reproduce the analysis."
