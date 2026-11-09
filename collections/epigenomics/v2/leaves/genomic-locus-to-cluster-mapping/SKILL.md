---
name: genomic-locus-to-cluster-mapping
description: Use when a BHi-Cect tree output (nested cluster list / named loci lists)
  must be joined with gene coordinates and per-locus ChIP-seq signal (e.g. RNAP2 in
  IMR90, H3K4me3 in K562) to produce per-enclave gene rankings for enrichment analysis.
when_to_use_negative:
- When the goal is contiguous topologically associating domain (TAD) calling — BHi-Cect
  clusters loci without imposing genomic contiguity, so results differ from TAD-based
  methods
- When Hi-C data is a dense contact matrix rather than the 3-column sparse format
  expected by Rao_matrix
- When mechanistic inference (e.g. loop extrusion vs phase separation) is sought —
  enclave-epigenomic association is correlative and such details 'cannot be inferred
  with the present analysis'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3912
- http://edamontology.org/topic_3169
tools:
- name: BHi-Cect
  role: Produces the nested cluster list / tree whose bottom enclaves receive the
    mapped gene loci
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Runtime environment in which the BHi-Cect functions are copy-pasted and the
    mapping/aggregation is executed
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Aggregates mean ChIP-seq signal per bottom enclave per cell line after locus
    mapping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:efe85b4332c7d35d134fc9cf69dc35dafbf9ac157a9b20bb701057dd2fea3486
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3912
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Genomic locus-to-cluster mapping

## Summary

Maps each gene's genomic locus onto the bottom enclaves of a BHi-Cect tree (nested cluster list of Hi-C loci) and aggregates mean ChIP-seq signal per enclave, enabling downstream per-enclave gene ranking and GSEA.

## When to use

Use when a BHi-Cect tree output (nested cluster list / named loci lists) must be joined with gene coordinates and per-locus ChIP-seq signal (e.g. RNAP2 in IMR90, H3K4me3 in K562) to produce per-enclave gene rankings for enrichment analysis.

## When NOT to use

- When the goal is contiguous topologically associating domain (TAD) calling — BHi-Cect clusters loci without imposing genomic contiguity, so results differ from TAD-based methods
- When Hi-C data is a dense contact matrix rather than the 3-column sparse format expected by Rao_matrix
- When mechanistic inference (e.g. loop extrusion vs phase separation) is sought — enclave-epigenomic association is correlative and such details 'cannot be inferred with the present analysis'

## Inputs

- BHi-Cect tree output: nested list defining the cluster hierarchy and a named list of loci per cluster (from spec_bipart on a power-transformed sparse Hi-C matrix)
- ChIP-seq signal data per genomic bin (RNAP2 for IMR90, H3K4me3 for K562)
- Gene genomic locus/bin coordinates for mapping genes to enclaves

## Outputs

- Gene-to-bottom-enclave assignment table
- Per-enclave mean ChIP-seq signal per cell line
- Per-enclave ranked gene lists (ranked by RNAP2 or H3K4me3 prevalence)
- FDR-corrected GSEA enrichment scores and gene-set hits written to gsea_results.csv

## How to apply

Run BHi-Cect on the 3-column sparse Hi-C matrix (Rao_matrix, then pow_trans with exponent 1, then graph construction and spec_bipart) to obtain the nested list defining the tree and the named list of loci per cluster. Identify the bottom enclaves — the deepest, mostly non-contiguous, interwoven clusters in the tree — from the nested cluster list. Map each gene's genomic locus (bin) to its containing bottom enclave using the named loci lists, then use dplyr in R to aggregate mean ChIP-seq signal (RNAP2 for IMR90, H3K4me3 for K562) per enclave per cell line. Rank genes by aggregated signal intensity within each bottom enclave, producing one ranked gene list per enclave, which can then feed GSEA with GO gene sets and Benjamini-Hochberg FDR correction. Correct application is judged by whether every gene resolves to exactly one bottom enclave and whether reproduced enrichment categories match the reported ones (skin, cytoskeleton/structural molecule activity, DNA packaging for IMR90 RNAP2; immune functions for K562 H3K4me3).

## Related tools

- **BHi-Cect** (Produces the nested cluster list / tree whose bottom enclaves receive the mapped gene loci) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Runtime environment in which the BHi-Cect functions are copy-pasted and the mapping/aggregation is executed) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Aggregates mean ChIP-seq signal per bottom enclave per cell line after locus mapping)

## Examples

```
chr_dat<- read_delim('./path/to/hic_file.txt'); chr_mat<-Rao_matrix(chr_dat); chr_pow<- pow_trans(chr_mat,1); g_chr1<- graph_from_adjacency_matrix(chr_pow,weighted = T); chr_spec_res<- spec_bipart(chr_pow ,g_chr1)
```

## Evaluation signals

- Every gene locus resolves to exactly one bottom enclave in the named loci lists; no unmapped genes remain after the join
- Mean ChIP-seq aggregation per enclave yields finite signal values for both IMR90 RNAP2 and K562 H3K4me3 tracks
- GSEA with GO gene sets on RNAP2 (IMR90) rankings recovers skin function, structural molecule activity (cytoskeleton) and DNA packaging enrichment
- GSEA on H3K4me3 (K562) rankings recovers immune function enrichment
- Benjamini-Hochberg FDR correction is applied across all tested GO gene sets and results are written to gsea_results.csv

## Limitations

- No dataset accession IDs or named data repositories are given for the IMR90 and K562 cell lines, so input ChIP-seq/Hi-C files must be sourced externally
- No software version is stated for BHi-Cect or the analysis software
- Cross-tissue variability of enclave arrangement is not examined and is flagged as future work
- The correspondence between enclave nesting and epigenomic activity is associative; mechanistic molecular dynamics details cannot be inferred

## Evidence

- [readme] Ouputs a list with 2 elements - a nested list defining the tree representing the hierarchical structure of the input matrix - a namesd list specifying the loci contained by all the clusters found by BHi-Cect: "a nested list defining the tree representing the hierarchical structure of the input matrix"
- [discussion] GSEA on bottom-enclave epigenomic prevalence: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [discussion] Enclaves as non-contiguous interwoven clusters: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [abstract] Nesting associated with epigenomic activity: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [readme] R functions usage: "chr_spec_res<- spec_bipart(chr_pow ,g_chr1)"
