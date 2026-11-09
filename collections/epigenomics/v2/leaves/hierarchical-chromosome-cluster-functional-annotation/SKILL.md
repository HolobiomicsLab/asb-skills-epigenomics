---
name: hierarchical-chromosome-cluster-functional-annotation
description: Use when you have a BHi-Cect nested cluster list plus ChIP-seq tracks
  and want to know which GO biological functions are enriched among genes whose loci
  fall into specific bottom enclaves.
when_to_use_negative:
- You need mechanistic details of chromatin dynamics (e.g. loop extrusion vs phase
  separation) — the analysis cannot infer these
- You want TAD-style contiguous-domain calling — BHi-Cect enclaves are mostly non-contiguous
  and interwoven, so this skill annotates clusters, not contiguity-bounded TADs
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0204
tools:
- name: BHi-Cect
  role: Produces the nested hierarchical cluster tree and bottom-enclave locus assignments
    from Hi-C data that are the unit of annotation
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: 'Host language: copy-paste BHi-Cect functions into the R session to run the
    clustering and downstream aggregation'
- name: dplyr
  role: Aggregates mean ChIP-seq signal per enclave and per cell line when mapping
    gene loci to bottom enclaves
- name: RSpectra
  role: Supports the spectral clustering step (spec_bipart) that yields the enclave
    hierarchy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:b563282e0c0774a0a926f6ee15179c7e49581732c98b50f84a72f2d205abe04f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - RSpectra
  - dplyr
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hierarchical-chromosome-cluster-functional-annotation

## Summary

Annotates BHi-Cect chromosome hierarchy clusters (bottom enclaves) functionally by aggregating ChIP-seq signal (RNAP2 in IMR90, H3K4me3 in K562) per enclave, ranking genes, and running GO gene-set GSEA with Benjamini-Hochberg FDR correction.

## When to use

Use when you have a BHi-Cect nested cluster list plus ChIP-seq tracks and want to know which GO biological functions are enriched among genes whose loci fall into specific bottom enclaves.

## When NOT to use

- You need mechanistic details of chromatin dynamics (e.g. loop extrusion vs phase separation) — the analysis cannot infer these
- You want TAD-style contiguous-domain calling — BHi-Cect enclaves are mostly non-contiguous and interwoven, so this skill annotates clusters, not contiguity-bounded TADs

## Inputs

- BHi-Cect tree output: nested cluster list and named list of loci per cluster (from spec_bipart)
- RNAP2 ChIP-seq signal data for IMR90
- H3K4me3 ChIP-seq signal data for K562
- Gene-to-genomic-locus mapping
- Gene Ontology (GO) gene sets for GSEA

## Outputs

- gsea_results.csv with FDR-corrected enrichment scores and gene-set hits per bottom enclave
- Per-enclave ranked gene lists by ChIP-seq prevalence (RNAP2 IMR90, H3K4me3 K562)

## How to apply

Load the BHi-Cect tree output (a nested list plus a named list of loci per cluster) and the 3-column sparse Hi-C data processed via Rao_matrix, pow_trans, graph building, and spec_bipart spectral clustering. Map each gene's genomic locus to its bottom enclave and aggregate mean ChIP-seq signal per enclave with dplyr, separately for RNAP2 (IMR90) and H3K4me3 (K562). Rank genes by signal intensity within each bottom enclave and run GSEA with Gene Ontology gene sets on each ranked list. Apply Benjamini-Hochberg FDR correction across all tested GO sets, then write FDR-corrected enrichment scores and gene-set hits to gsea_results.csv. Correctness is judged by reproducing the reported category pattern: skin function, structural molecule activity (cytoskeleton) and DNA packaging for IMR90 RNAP2; immune functions for K562 H3K4me3.

## Related tools

- **BHi-Cect** (Produces the nested hierarchical cluster tree and bottom-enclave locus assignments from Hi-C data that are the unit of annotation) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Host language: copy-paste BHi-Cect functions into the R session to run the clustering and downstream aggregation)
- **dplyr** (Aggregates mean ChIP-seq signal per enclave and per cell line when mapping gene loci to bottom enclaves)
- **RSpectra** (Supports the spectral clustering step (spec_bipart) that yields the enclave hierarchy)

## Examples

```
chr_dat <- read_delim(./path/to/hic_file.txt); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res <- spec_bipart(chr_pow,g_chr1)
```

## Evaluation signals

- GSEA on IMR90 RNAP2 bottom-enclave prevalence recovers skin function, structural molecule activity (cytoskeleton) and DNA packaging categories
- GSEA on K562 H3K4me3 bottom-enclave prevalence recovers immune function categories
- Benjamini-Hochberg FDR correction has been applied across all tested GO gene sets before reporting
- gsea_results.csv exists and contains FDR-corrected enrichment scores plus gene-set hits
- Every gene's locus resolves to exactly one bottom enclave before signal aggregation

## Limitations

- No dataset accession IDs or named data repositories given for the IMR90 and K562 ChIP-seq data in the discussion section
- No software version stated for BHi-Cect or the analysis software
- Cross-tissue variability of enclave arrangement not examined; flagged as future work
- Biophysical constraints from in vitro and molecular dynamics studies and DNA binding factor content not yet integrated into chromosome segmentation

## Evidence

- [discussion] GSEA of bottom-enclave epigenomic prevalence: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [discussion] Reported enrichment categories for IMR90 and K562: "cytoskeleton and skin function enrichment in IMR90 fibroblast samples or immune function enrichment in K562 leukemia samples"
- [discussion] Enclave clusters are non-contiguous and interwoven: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [readme] Required R packages for the pipeline: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [readme] BHi-Cect output structure used to map loci to enclaves: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
