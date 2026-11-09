---
name: chromosome-cluster-nesting-analysis
description: Use when you have a 3-column sparse Hi-C contact matrix (e.g. IMR90 at
  50 kb or 5 kb resolution) and need a recursive, hierarchy-aware clustering of loci
  without imposing genomic contiguity.
when_to_use_negative:
- When the goal is standard contiguous TAD calling, since TADs postulate clustering
  among neighboring loci contiguous along the genome coordinate and BHi-Cect instead
  allows non-contiguous, interwoven enclaves
- When the input is not a 3-column sparse Hi-C contact matrix but an already-processed
  feature table or a non-Hi-C interaction dataset
- When mechanistic inference about loop extrusion vs phase separation is required
  — the analysis cannot infer these details
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: BHi-Cect
  role: 'Core algorithm: Rao_matrix, pow_trans, and spec_bipart functions convert
    sparse contacts into a matrix, power-transform them, and recursively spectral-bipartition
    the interaction graph'
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: readr
  role: Loads the 3-column sparse matrix Hi-C data via read_delim
- name: igraph
  role: Builds the weighted Hi-C interaction graph with graph_from_adjacency_matrix(weighted=TRUE)
    before spectral clustering
- name: Matrix
  role: Sparse matrix format for the converted contact matrix
- name: RSpectra
  role: Spectral computations backing the recursive bipartitioning
- name: caret
  role: Listed as a required R package for the BHi-Cect workflow
- name: dplyr
  role: Listed as a required R package for data manipulation
- name: data.tree
  role: Listed as a required R package supporting the nested tree representation
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:4d16a13babcf84d9515cc4e2fee7041bcce395bdabd8c6e3b6aa10198340c4a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - Matrix
  - R
  - RSpectra
  - caret
  - data.tree
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# chromosome-cluster-nesting-analysis

## Summary

Runs BHi-Cect, a top-down spectral clustering method that partitions Hi-C loci into nested, non-contiguous self-interacting clusters ('enclaves') and returns a binary partition tree plus per-cluster locus lists.

## When to use

Use when you have a 3-column sparse Hi-C contact matrix (e.g. IMR90 at 50 kb or 5 kb resolution) and need a recursive, hierarchy-aware clustering of loci without imposing genomic contiguity.

## When NOT to use

- When the goal is standard contiguous TAD calling, since TADs postulate clustering among neighboring loci contiguous along the genome coordinate and BHi-Cect instead allows non-contiguous, interwoven enclaves
- When the input is not a 3-column sparse Hi-C contact matrix but an already-processed feature table or a non-Hi-C interaction dataset
- When mechanistic inference about loop extrusion vs phase separation is required — the analysis cannot infer these details

## Inputs

- 3-column sparse Hi-C contact matrix text file (locus_i, locus_j, contact count), e.g. IMR90 Hi-C at 50 kb or 5 kb resolution
- Optional: benchmark TAD and compartment annotation clusterings (e.g. Rao et al.) for VI significance testing
- Optional: epigenomic tracks (RNAP2, H3K4me3, TF content) for enriching bottom enclaves

## Outputs

- A nested list (binary partition tree) defining the hierarchical structure of the input contact matrix
- A named list specifying the loci contained in every cluster (enclave) found by BHi-Cect
- Optionally: VI values, null distribution from 500 random recursive bisections, and empirical P-value (e.g. vi_significance_results.csv); saved RDS such as bhi_cect_imr90_clusters.rds

## How to apply

Load the 3-column sparse Hi-C contact file with readr::read_delim, then convert the triplets into a sparse matrix with BHi-Cect's Rao_matrix function and apply the power transformation (e.g. pow_trans(chr_mat,1), exponent 1). Build the weighted Hi-C interaction graph with igraph::graph_from_adjacency_matrix(weighted=TRUE) and run the recursive top-down spectral bipartition via BHi-Cect's spec_bipart function to obtain the nested cluster tree and the named list of loci per cluster. Because BHi-Cect clusters loci 'considering every locus with no assumption of genomic contiguity', assess non-contiguity by checking whether member loci of returned clusters interleave along the genome coordinate (plaid-pattern partitioning) rather than forming contiguous blocks. At 50 kb resolution BHi-Cect highlights global clusters spanning the whole chromosome (top-level enclaves average ~120 Mb span), while 5 kb data focuses on local clusters under 1 Mb. Downstream, cluster agreement with Rao et al. TAD/compartment annotations can be tested with the variation of information metric against ~500 random recursive bisection nulls (empirical P < 0.01), and the nesting of local clusters within global enclaves can be related to epigenomic activity of the underlying DNA.

## Related tools

- **BHi-Cect** (Core algorithm: Rao_matrix, pow_trans, and spec_bipart functions convert sparse contacts into a matrix, power-transform them, and recursively spectral-bipartition the interaction graph) — https://github.com/princeps091-binf/BHi-Cect
- **readr** (Loads the 3-column sparse matrix Hi-C data via read_delim)
- **igraph** (Builds the weighted Hi-C interaction graph with graph_from_adjacency_matrix(weighted=TRUE) before spectral clustering)
- **Matrix** (Sparse matrix format for the converted contact matrix)
- **RSpectra** (Spectral computations backing the recursive bipartitioning)
- **caret** (Listed as a required R package for the BHi-Cect workflow)
- **dplyr** (Listed as a required R package for data manipulation)
- **data.tree** (Listed as a required R package supporting the nested tree representation)

## Examples

```
chr_dat <- read_delim('hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow, weighted = T); chr_spec_res <- spec_bipart(chr_pow, g_chr1)
```

## Evaluation signals

- The saved output RDS must contain both the nested tree element and the named cluster-loci list (a two-element list per the README)
- Detected clusters should be mostly non-contiguous: member loci must interleave along the genome coordinate (plaid-pattern regions visible in the Hi-C heatmap)
- Top-level enclave genomic spans should be consistent with the reported ~120 Mb average when using 50 kb resolution data
- Agreement with Rao et al. TAD/compartment calls via variation of information must yield an empirical P-value below 0.01 against 500 random recursive bisection nulls
- With 5 kb resolution data, clusters should be local and span less than 1 Mb

## Limitations

- Implementation as a stand-alone package is ongoing; users must copy-paste the BHi-Cect script functions into an R session
- Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from the present analysis
- Cross-tissue variability of enclave arrangement was not examined and is flagged as future work
- Depending on Hi-C resolution the method highlights different scales: global clusters at 50 kb vs local clusters under 1 Mb at 5 kb

## Evidence

- [abstract] BHi-Cect, a novel top-down algorithm that finds clusters by considering every locus with no assumption of genomic contiguity using spectral clustering: "Here, we present BHi-Cect, a novel top-down algorithm that finds clusters by considering every locus with no assumption of genomic contiguity using spectral clustering."
- [methods] chr_mat<-Rao_matrix(chr_dat): "chr_mat<-Rao_matrix(chr_dat)"
- [methods] chr_spec_res<- spec_bipart(chr_pow ,g_chr1): "chr_spec_res<- spec_bipart(chr_pow ,g_chr1)"
- [readme] Ouputs a list with 2 elements: "- Ouputs a list with 2 elements 
  - a nested list defining the tree representing the hierarchical structure of the input matrix"
- [discussion] enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [discussion] with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome: "with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb)"
