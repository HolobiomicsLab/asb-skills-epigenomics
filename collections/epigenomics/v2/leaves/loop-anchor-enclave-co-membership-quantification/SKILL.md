---
name: loop-anchor-enclave-co-membership-quantification
description: Use when comparing how well a chromosome clustering scheme (e.g., BHi-Cect
  bottom enclaves) versus TAD segmentations contains pairs of DNA loop anchor sites
  from Hi-C data at a fixed binning resolution.
when_to_use_negative:
- When the Hi-C data resolution differs from 5 kb binning — the article notes that
  with low resolution (50 kb) data BHi-Cect highlights global chromosome clusters
  rather than the local clusters relevant to loop anchor integration
- When the goal is to identify TAD boundaries by contiguity rather than to evaluate
  a clustering's co-membership of loop anchors, since this skill assumes pre-computed
  cluster memberships and pre-called loop anchors
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: BHi-Cect
  role: Top-down spectral clustering of Hi-C loci producing the enclave hierarchy
    whose bottom-level clusters are tested for loop anchor co-membership
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: readr
  role: Loads the 3-column sparse Hi-C contact matrix file
- name: Matrix
  role: Sparse matrix representation of the Hi-C contact matrix via Rao_matrix
- name: igraph
  role: Builds the weighted Hi-C interaction graph (graph_from_adjacency_matrix) fed
    into spectral clustering
- name: RSpectra
  role: Eigendecomposition backend for the recursive bipartite spectral clustering
    in spec_bipart
- name: dplyr
  role: Data manipulation for mapping loop anchors and TADs onto 5 kb bins and counting
    co-membership
- name: data.tree
  role: Handles the nested tree representation of the BHi-Cect cluster hierarchy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:082533659d99b073d7ea584ed851d9c7d6191d7dc4ea48cc9a0201015a52493f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - Matrix
  - R
  - RSpectra
  - data.tree
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# loop-anchor-enclave-co-membership-quantification

## Summary

Quantifies how well non-contiguous chromosome clusters (bottom enclaves) from BHi-Cect integrate DNA loops versus TADs, by measuring the percentage of HICCUPS loop anchor-site pairs falling within the same bottom enclave versus the same TAD at 5 kb resolution.

## When to use

Use when comparing how well a chromosome clustering scheme (e.g., BHi-Cect bottom enclaves) versus TAD segmentations contains pairs of DNA loop anchor sites from Hi-C data at a fixed binning resolution.

## When NOT to use

- When the Hi-C data resolution differs from 5 kb binning — the article notes that with low resolution (50 kb) data BHi-Cect highlights global chromosome clusters rather than the local clusters relevant to loop anchor integration
- When the goal is to identify TAD boundaries by contiguity rather than to evaluate a clustering's co-membership of loop anchors, since this skill assumes pre-computed cluster memberships and pre-called loop anchors

## Inputs

- IMR90 5 kb Hi-C contact matrix in 3-column sparse text format (chr_dat)
- HICCUPS DNA loop anchor-site coordinates
- TAD segmentation coordinates
- BHi-Cect R script functions (Rao_matrix, pow_trans, spec_bipart)

## Outputs

- Percentage of DNA loop anchor-site pairs within the same bottom enclave (reported 79.9%)
- Percentage of DNA loop anchor-site pairs within the same TAD (reported 59.9%)
- anchor_overlap_comparison.csv containing the two percentages

## How to apply

Load the IMR90 5 kb Hi-C contact matrix in 3-column sparse format with readr, convert it via Rao_matrix, apply the power transform pow_trans with exponent 1, build the weighted interaction graph with igraph, and run recursive bipartite spectral clustering (spec_bipart) to obtain the BHi-Cect cluster hierarchy. Parse the spec_bipart output — a nested tree list plus a named list of loci per cluster — to extract bottom-level enclave memberships. Map the HICCUPS loop anchor coordinates and TAD segmentations onto the same 5 kb binning so each anchor-site pair is annotated with both its bottom enclave and its TAD. Count the fraction of anchor pairs whose two loci share the same bottom enclave and, separately, the same TAD, expressing both as percentages. The rationale is that enclaves are mostly non-contiguous, deeply interwoven clusters, so their ability to co-localize loop anchors tests whether they capture regulatory architecture better than contiguous TADs.

## Related tools

- **BHi-Cect** (Top-down spectral clustering of Hi-C loci producing the enclave hierarchy whose bottom-level clusters are tested for loop anchor co-membership) — https://github.com/princeps091-binf/BHi-Cect
- **readr** (Loads the 3-column sparse Hi-C contact matrix file)
- **Matrix** (Sparse matrix representation of the Hi-C contact matrix via Rao_matrix)
- **igraph** (Builds the weighted Hi-C interaction graph (graph_from_adjacency_matrix) fed into spectral clustering)
- **RSpectra** (Eigendecomposition backend for the recursive bipartite spectral clustering in spec_bipart)
- **dplyr** (Data manipulation for mapping loop anchors and TADs onto 5 kb bins and counting co-membership)
- **data.tree** (Handles the nested tree representation of the BHi-Cect cluster hierarchy)

## Examples

```
chr_dat<-read_delim('hic_file.txt'); chr_mat<-Rao_matrix(chr_dat); chr_pow<-pow_trans(chr_mat,1); g<-graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res<-spec_bipart(chr_pow,g)
```

## Evaluation signals

- The percentage of same-bottom-enclave anchor pairs should be 79.9% +/- 0.5 percentage points
- The percentage of same-TAD anchor pairs should be 59.9% +/- 0.5 percentage points
- Both percentages must be computed at the same 5 kb resolution so the enclave vs TAD comparison is apples-to-apples
- anchor_overlap_comparison.csv should be produced and contain the two computed percentages
- spec_bipart output must contain both a nested tree list and a named list of loci per cluster before memberships can be extracted

## Limitations

- TAD methods postulate clustering among neighboring loci contiguous along the genome coordinate, which can mask more complex interaction patterns seen in high resolution Hi-C data; the comparison is therefore biased toward the non-contiguous enclave representation
- Implementation as a stand-alone package was ongoing at time of publication — users must copy-paste functions from the BHi-Cect script into an R session
- The article does not state software versions for BHi-Cect or any analysis software, so exact reproducibility may vary

## Evidence

- [intro] TADs postulate clustering among neighboring loci that are contiguous along the genome coordinate, which can mask more complex interaction patterns often seen in more recent high resolution Hi-C data: "TADs postulate clustering among neighboring loci that are contiguous along the genome coordinate, which can mask more complex interaction patterns"
- [discussion] Enclaves as non-contiguous clusters: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [readme] README usage pipeline: "#load the 3-column sparse matrix Hi-C data
  chr_dat<- read_delim(./path/to/hic_file.txt)
  #convert into sparse matrix format
  chr_mat<-Rao_matrix(chr_dat)"
- [readme] README output format: "Ouputs a list with 2 elements 
  - a nested list defining the tree representing the hierarchical structure of the input matrix
  - a namesd list specifying the loci contained by all the clusters"
