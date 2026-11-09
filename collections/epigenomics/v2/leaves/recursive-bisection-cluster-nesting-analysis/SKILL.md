---
name: recursive-bisection-cluster-nesting-analysis
description: Use when you have a 3-column sparse Hi-C contact matrix (e.g. IMR90)
  and need a multi-scale, nested partition tree of loci clusters without assuming
  genomic contiguity.
when_to_use_negative:
- When the analysis goal assumes clusters of neighboring loci contiguous along the
  genome coordinate (TAD-style segmentation), since BHi-Cect deliberately imposes
  no genomic contiguity and can return non-contiguous, interwoven enclaves
- When input is not raw Hi-C contact data (e.g. an already-computed feature table,
  expression matrix, or pre-called TAD domains) — the skill expects a 3-column sparse
  contact matrix
- When mechanistic inference about chromatin folding (e.g. loop extrusion vs phase
  separation) is the goal — the authors state such details cannot be inferred with
  this analysis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: BHi-Cect
  role: Core top-down spectral bisection algorithm producing the BHi-Cect partition
    tree (BPT) from Hi-C contact data
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: readr
  role: Loads the 3-column sparse matrix Hi-C data (read_delim)
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: Matrix
  role: Provides sparse matrix format; Rao_matrix converts contact data into sparse
    matrix form
- name: igraph
  role: Builds the weighted Hi-C interaction graph via graph_from_adjacency_matrix
- name: RSpectra
  role: Eigendecomposition of the random-walk normalized Laplacian inside spec_bipart
- name: data.tree
  role: Aggregates recursive partitions into the BHi-Cect partition tree
- name: R
  role: Runtime environment; BHi-Cect functions are copy-pasted into the R session
  repo: https://github.com/princeps091-binf/BHi-Cect
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:1ec2bb7b1f3c5ebc69f5b28b4b2a6ca8f7ab4b16ef5ad5ab2c523fa7ba101660
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# recursive-bisection-cluster-nesting-analysis

## Summary

Runs BHi-Cect, a top-down spectral clustering algorithm that recursively bisects a Hi-C contact matrix (via the Fiedler vector of the normalized Laplacian and k-means with 5 seedings) into a BHi-Cect partition tree (BPT) of nested, non-contiguous chromosome clusters called enclaves.

## When to use

Use when you have a 3-column sparse Hi-C contact matrix (e.g. IMR90) and need a multi-scale, nested partition tree of loci clusters without assuming genomic contiguity.

## When NOT to use

- When the analysis goal assumes clusters of neighboring loci contiguous along the genome coordinate (TAD-style segmentation), since BHi-Cect deliberately imposes no genomic contiguity and can return non-contiguous, interwoven enclaves
- When input is not raw Hi-C contact data (e.g. an already-computed feature table, expression matrix, or pre-called TAD domains) — the skill expects a 3-column sparse contact matrix
- When mechanistic inference about chromatin folding (e.g. loop extrusion vs phase separation) is the goal — the authors state such details cannot be inferred with this analysis

## Inputs

- 3-column sparse Hi-C contact matrix text file (e.g. IMR90 chr17/chr18/chr22)
- Power-transform exponent for pow_trans (used value: 1)
- R session with functions from the BHi-Cect script pasted in

## Outputs

- List with 2 elements: a nested list defining the tree of the hierarchical structure (BPT) and a named list specifying the loci contained in all found clusters
- bhi_cect_partition_tree.rds export of the nested tree structure and cluster loci lists
- Per-bisection classifications as split or strip via the expansion metric

## How to apply

Load the 3-column sparse Hi-C contact file with readr and convert it to a sparse matrix using Rao_matrix (Matrix package). Apply the power transform pow_trans with exponent 1 to the sparse contact matrix, then build the weighted Hi-C interaction graph with igraph's graph_from_adjacency_matrix (weighted=TRUE). Run spec_bipart, which performs an RSpectra eigendecomposition of the random-walk normalized Laplacian and applies k-means with k=2 over 5 different seedings to reach a consensus optimal bisection. Recursively bisect each resulting submatrix, classifying each division as a split or strip via the expansion metric, and aggregate the recursive partitions into a BHi-Cect partition tree using data.tree. Export the tree and named list of loci per cluster as bhi_cect_partition_tree.rds; resolution of detected clusters depends on Hi-C bin size (50 kb data yields global clusters, 5 kb data yields local clusters under 1 Mb).

## Related tools

- **BHi-Cect** (Core top-down spectral bisection algorithm producing the BHi-Cect partition tree (BPT) from Hi-C contact data) — https://github.com/princeps091-binf/BHi-Cect
- **readr** (Loads the 3-column sparse matrix Hi-C data (read_delim)) — https://github.com/princeps091-binf/BHi-Cect
- **Matrix** (Provides sparse matrix format; Rao_matrix converts contact data into sparse matrix form)
- **igraph** (Builds the weighted Hi-C interaction graph via graph_from_adjacency_matrix)
- **RSpectra** (Eigendecomposition of the random-walk normalized Laplacian inside spec_bipart)
- **data.tree** (Aggregates recursive partitions into the BHi-Cect partition tree)
- **R** (Runtime environment; BHi-Cect functions are copy-pasted into the R session) — https://github.com/princeps091-binf/BHi-Cect

## Examples

```
chr_dat<-read_delim('./path/to/hic_file.txt'); chr_mat<-Rao_matrix(chr_dat); chr_pow<-pow_trans(chr_mat,1); g_chr1<-graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res<-spec_bipart(chr_pow,g_chr1)
```

## Evaluation signals

- Output list contains exactly 2 elements: a nested list defining the tree of the hierarchical structure and a named list of loci per cluster
- Every child cluster's loci are nested within its parent cluster (hierarchy invariant of the BPT)
- Each bisection is classified as split or strip via the expansion metric, and consensus k-means over 5 seedings yields the same optimal bisection
- Spec_bipart uses k=2 clustering on the second eigenvector (Fiedler vector) of the normalized Laplacian — verify the second eigenvector, not a later one, drives the split
- Input contact matrix was power-transformed (pow_trans with exponent 1) before graph construction

## Limitations

- No software version stated for BHi-Cect or any analysis software; implementation as a stand-alone package is ongoing (functions must be copy-pasted into the R session)
- Resolution is data-dependent: with low resolution Hi-C data (50 kb) BHi-Cect highlights global chromosome clusters, while with high resolution data (5 kb) it focuses on local clusters spanning less than 1 Mb
- Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from the present analysis
- Cross-tissue variability of enclave arrangement not examined; flagged as future work

## Evidence

- [abstract] Here, we present BHi-Cect, a novel top-down algorithm that finds clusters by considering every locus with no assumption of genomic contiguity using spectral clustering.: "Here, we present BHi-Cect, a novel top-down algorithm that finds clusters by considering every locus with no assumption of genomic contiguity using spectral clustering."
- [methods] chr_mat<-Rao_matrix(chr_dat): "chr_mat<-Rao_matrix(chr_dat)"
- [methods] chr_pow<- pow_trans(chr_mat,1): "chr_pow<- pow_trans(chr_mat,1)"
- [methods] chr_spec_res<- spec_bipart(chr_pow ,g_chr1): "chr_spec_res<- spec_bipart(chr_pow ,g_chr1)"
- [other] Each BHi-Cect bisection uses k-means clustering with k = 2 applied to the second eigenvector (Fiedler vector) of the normalized Laplacian, collecting the consensus from five different seedings: [evidence span withheld — anchor status: not_found]
- [readme] Ouputs a list with 2 elements - a nested list defining the tree representing the hierarchical structure of the input matrix: "- Ouputs a list with 2 elements 
  - a nested list defining the tree representing the hierarchical structure of the input matrix
  - a namesd list specifying the loci contained by all the clusters"
- [discussion] with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb): "with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb)"
