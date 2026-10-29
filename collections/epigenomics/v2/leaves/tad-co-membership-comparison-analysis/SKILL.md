---
name: tad-co-membership-comparison-analysis
description: Use when you have a BHi-Cect cluster hierarchy plus a TAD segmentation
  and loop anchor coordinates at matched 5 kb binning, and need to compare co-membership
  of anchor-site pairs across the two partitions.
when_to_use_negative:
- The Hi-C data is low resolution (e.g. 50 kb), where BHi-Cect highlights global chromosome
  clusters rather than the local sub-Mb clusters relevant to loop anchor integration
- You need contiguous genomic domains only — enclaves are mostly non-contiguous and
  interwoven by design, so this comparison is inappropriate if contiguity is a hard
  requirement
- Loop anchors and TADs cannot be mapped onto the same binning resolution as the Hi-C
  contact matrix (e.g. mismatched 5 kb binning)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: BHi-Cect
  role: Top-down spectral clustering algorithm producing the enclave hierarchy whose
    bottom clusters are compared against TADs
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Runtime environment for the BHi-Cect script functions and downstream co-membership
    counting
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: readr
  role: Loads the 3-column sparse matrix Hi-C data (read_delim)
- name: igraph
  role: Builds the weighted Hi-C interaction graph via graph_from_adjacency_matrix
    before spectral clustering
- name: RSpectra
  role: Eigendecomposition backend for the recursive bipartite spectral clustering
    (spec_bipart)
- name: Matrix
  role: Sparse matrix representation of the contact matrix after conversion with Rao_matrix
- name: dplyr
  role: Data manipulation for annotating anchor-site pairs with enclave and TAD co-membership
- name: data.tree
  role: Handling the nested tree representation of the BHi-Cect cluster hierarchy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:5df01bf4e91238776b38c66beb2879933f2c85b6880476da59f0981da9355c79
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# tad-co-membership-comparison-analysis

## Summary

Quantifies how well BHi-Cect bottom enclaves versus TAD segmentations integrate DNA loops by computing the percentage of HICCUPS loop anchor-site pairs whose two loci fall in the same cluster under each partitioning of the IMR90 5 kb Hi-C data.

## When to use

Use when you have a BHi-Cect cluster hierarchy plus a TAD segmentation and loop anchor coordinates at matched 5 kb binning, and need to compare co-membership of anchor-site pairs across the two partitions.

## When NOT to use

- The Hi-C data is low resolution (e.g. 50 kb), where BHi-Cect highlights global chromosome clusters rather than the local sub-Mb clusters relevant to loop anchor integration
- You need contiguous genomic domains only — enclaves are mostly non-contiguous and interwoven by design, so this comparison is inappropriate if contiguity is a hard requirement
- Loop anchors and TADs cannot be mapped onto the same binning resolution as the Hi-C contact matrix (e.g. mismatched 5 kb binning)

## Inputs

- IMR90 5 kb Hi-C contact matrix in 3-column sparse text format
- HICCUPS DNA loop anchor coordinates (genomic intervals)
- TAD segmentation coordinates for IMR90
- BHi-Cect R functions (Rao_matrix, pow_trans, graph_from_adjacency_matrix, spec_bipart)

## Outputs

- Bottom-level enclave memberships per locus (nested tree list plus named list of loci per cluster from spec_bipart)
- Percentage of loop anchor-site pairs within the same bottom enclave (expected ~79.9%)
- Percentage of loop anchor-site pairs within the same TAD (expected ~59.9%)
- anchor_overlap_comparison.csv containing the two percentages

## How to apply

Load the IMR90 5 kb Hi-C contact matrix in 3-column sparse format with readr, convert it with Rao_matrix, apply pow_trans with exponent 1, build the weighted igraph interaction graph, and run spec_bipart to obtain the BHi-Cect cluster hierarchy. Parse the spec_bipart output (a nested tree list plus a named list of loci per cluster) to extract bottom-level enclave memberships. Map HICCUPS loop anchor coordinates and the TAD segmentation onto the same 5 kb binning so each anchor-site pair is annotated with both its bottom enclave and its TAD. Count the fraction of anchor pairs whose two loci share the same bottom enclave and, separately, the same TAD, express both as percentages, and write them to anchor_overlap_comparison.csv. The rationale is that enclaves are non-contiguous, interwoven clusters, so they may integrate loops that contiguous TADs miss.

## Related tools

- **BHi-Cect** (Top-down spectral clustering algorithm producing the enclave hierarchy whose bottom clusters are compared against TADs) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Runtime environment for the BHi-Cect script functions and downstream co-membership counting) — https://github.com/princeps091-binf/BHi-Cect
- **readr** (Loads the 3-column sparse matrix Hi-C data (read_delim))
- **igraph** (Builds the weighted Hi-C interaction graph via graph_from_adjacency_matrix before spectral clustering)
- **RSpectra** (Eigendecomposition backend for the recursive bipartite spectral clustering (spec_bipart))
- **Matrix** (Sparse matrix representation of the contact matrix after conversion with Rao_matrix)
- **dplyr** (Data manipulation for annotating anchor-site pairs with enclave and TAD co-membership)
- **data.tree** (Handling the nested tree representation of the BHi-Cect cluster hierarchy)

## Examples

```
chr_dat<-read_delim('IMR90_5kb.txt'); chr_mat<-Rao_matrix(chr_dat); chr_pow<-pow_trans(chr_mat,1); g<-graph_from_adjacency_matrix(chr_pow,weighted=T); res<-spec_bipart(chr_pow,g)
```

## Evaluation signals

- Same-bottom-enclave percentage reproduces the reported 79.9% within +/-0.5 percentage points
- Same-TAD percentage reproduces the reported 59.9% within +/-0.5 percentage points at the same 5 kb resolution
- anchor_overlap_comparison.csv exists and contains exactly the two percentages as columns/values
- Every loop anchor-site pair is annotated with exactly one bottom enclave and one TAD after 5 kb binning (no unannotated pairs)
- spec_bipart output parses into both a nested tree list and a named list of loci per cluster covering all loci of the chromosome

## Limitations

- TAD-based methods postulate clustering among contiguous neighboring loci, which can mask more complex interaction patterns; the enclave comparison is only meaningful for partitions from non-contiguous clustering
- The bottom enclaves versus TADs comparison does not by itself explain mechanism (e.g. loop extrusion vs phase separation), which cannot be inferred from the present analysis
- Mechanistic interpretation aside, results depend on the specific IMR90 5 kb data and HICCUPS loop calls; cross-tissue variability of enclave arrangement was not examined

## Evidence

- [other] Bottom enclaves contain 79.9% of DNA loop anchor-site pairs, integrating DNA loops better than TADs, which contain only 59.9% at the same 5 kb resolution.: [evidence span withheld — anchor status: not_found]
- [intro] TADs postulate clustering among neighboring loci that are contiguous along the genome coordinate, which can mask more complex interaction patterns: "TADs postulate clustering among neighboring loci that are contiguous along the genome coordinate, which can mask more complex interaction patterns often seen in more recent high resolution Hi-C data"
- [intro] we propose a new clustering method termed BHi-Cect. BHi-Cect clusters loci by considering all possible combinations without imposing contiguity: "we propose a new clustering method termed BHi-Cect. BHi-Cect clusters loci by considering all possible combinations without imposing contiguity along the genome coordinate using spectral clustering"
- [readme] chr_dat<- read_delim(./path/to/hic_file.txt): "#load the 3-column sparse matrix Hi-C data
  chr_dat<- read_delim(./path/to/hic_file.txt)"
- [methods] chr_spec_res<- spec_bipart(chr_pow ,g_chr1): "chr_spec_res<- spec_bipart(chr_pow ,g_chr1)"
- [discussion] A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
