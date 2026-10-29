---
name: hierarchical-nestedness-scoring
description: Use when you already have a BHi-Cect tree (nested list plus named locus-per-cluster
  list) and need to quantify each cluster's nested position and test whether nesting
  within global clusters associates with epigenomic activity (DNase, LAD, ChromHMM,
  CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1).
when_to_use_negative:
- Input Hi-C data has not yet been clustered by BHi-Cect — the NS score requires the
  BPT tree and cluster-loci lists as prerequisites
- You need contiguous, coordinate-adjacent segmentation such as TADs — enclaves are
  mostly non-contiguous, deeply interwoven clusters that TAD-based contiguity-assuming
  methods mask
- No epigenomic annotation tracks are available for the cell line, in which case only
  the NS scoring step (not the enrichment contrast) can be performed
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BHi-Cect
  role: Produces the nested tree list and named cluster-loci lists (BPT) that the
    NS scoring and enrichment analysis operate on
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Host environment; BHi-Cect functions are copy-pasted into the R session
- name: data.tree
  role: Parses the nested BHi-Cect hierarchy to identify top enclaves (global clusters)
    and bottom enclaves (terminal local clusters)
- name: readr
  role: Loads the 3-column sparse Hi-C matrix and required packages into the R session
- name: dplyr
  role: Data manipulation for intersecting enclave loci with epigenomic tracks and
    normalizing overlap counts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:7059dfa5e18afae79226b873022d235fbc000f4dbe98a8198200ca9c1a090e2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - data.tree
  - dplyr
  - readr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hierarchical-nestedness-scoring

## Summary

Computes a nestedness (NS) score for every cluster in a BHi-Cect partition tree (BPT) by anchoring top enclaves, intermediate nodes, and bottom enclaves/leaves at 0/0.5/1, then contrasts the NS score against epigenomic activity of the underlying DNA to reproduce the top- versus bottom-enclave antagonism.

## When to use

Use when you already have a BHi-Cect tree (nested list plus named locus-per-cluster list) and need to quantify each cluster's nested position and test whether nesting within global clusters associates with epigenomic activity (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1).

## When NOT to use

- Input Hi-C data has not yet been clustered by BHi-Cect — the NS score requires the BPT tree and cluster-loci lists as prerequisites
- You need contiguous, coordinate-adjacent segmentation such as TADs — enclaves are mostly non-contiguous, deeply interwoven clusters that TAD-based contiguity-assuming methods mask
- No epigenomic annotation tracks are available for the cell line, in which case only the NS scoring step (not the enrichment contrast) can be performed

## Inputs

- 3-column sparse Hi-C contact matrix (e.g., IMR90 chr17/chr18/chr22) processed via Rao_matrix, pow_trans, and spec_bipart
- BHi-Cect output: nested list defining the BPT tree plus named list specifying the loci contained by all clusters
- Epigenomic annotation tracks: DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1

## Outputs

- NS (nestedness) score per cluster, anchored 0/0.5/1 with interpolation for intermediate nodes
- enclave_nestedness_enrichment.csv containing the NS score column plus enrichment columns for all listed epigenomic marks
- Top- versus bottom-enclave contrast quantifying the epigenomic activity antagonism

## How to apply

Load the BHi-Cect output (a nested list defining the tree plus a named list of loci per cluster) in R using readr and dplyr, and parse the hierarchy with data.tree. Identify top enclaves as the large global clusters (which BHi-Cect surfaces with low-resolution 50 kb Hi-C data) and bottom enclaves as the terminal local clusters (spanning <1 Mb at 5 kb resolution). Assign anchors 0 to top enclaves, 1 to bottom enclaves/leaves, 0.5 to mid-level nodes, and interpolate NS scores for all intermediate nodes along the BPT. Intersect each enclave's loci with epigenomic annotation tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, and CTCF/RAD21/CHD1), counting overlaps per enclave, then normalize overlap counts per top enclave to obtain enrichment values per mark. Contrast top- versus bottom-enclave enrichment: the expected signature is progressive enrichment of epigenomic activity with increasing nestedness in top enclaves and progressive depletion in bottom enclaves, concentrating active regions at the top/bottom enclave boundary. Write per-cluster NS scores and per-mark enrichment values to enclave_nestedness_enrichment.csv.

## Related tools

- **BHi-Cect** (Produces the nested tree list and named cluster-loci lists (BPT) that the NS scoring and enrichment analysis operate on) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Host environment; BHi-Cect functions are copy-pasted into the R session)
- **data.tree** (Parses the nested BHi-Cect hierarchy to identify top enclaves (global clusters) and bottom enclaves (terminal local clusters))
- **readr** (Loads the 3-column sparse Hi-C matrix and required packages into the R session)
- **dplyr** (Data manipulation for intersecting enclave loci with epigenomic tracks and normalizing overlap counts)

## Examples

```
install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree')); chr_spec_res <- spec_bipart(chr_pow, g_chr1)
```

## Evaluation signals

- enclave_nestedness_enrichment.csv exists and contains an NS score column plus enrichment columns for all listed marks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1)
- Anchor assignments are correct: top enclaves coded 0, bottom enclaves/leaves coded 1, mid-level nodes coded 0.5
- Top-enclave enrichment values are normalized overlap counts (bounded, non-negative) with one value per mark per enclave
- Qualitative validation: epigenomic activity is progressively enriched with nestedness in top enclaves and progressively depleted in bottom enclaves, reproducing the reported antagonism concentrated at the top/bottom enclave boundary
- Every cluster in the BHi-Cect cluster list receives a finite NS score (no missing nodes in the interpolation)

## Limitations

- Resolution-dependent behavior: with low-resolution Hi-C (50 kb) BHi-Cect highlights global clusters, with high-resolution data (5 kb) it focuses on local clusters spanning less than 1 Mb, so the top/bottom enclave balance depends on input resolution
- Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from the present analysis
- Cross-tissue variability of enclave arrangement was not examined and is flagged as future work
- No software version stated for BHi-Cect or the analysis packages; implementation as a stand-alone package was still ongoing

## Evidence

- [abstract] We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA.: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [discussion] Our clustering method augments existing structural descriptions by using the BPT representation for chromosome structure: "Our clustering method augments existing structural descriptions by using the BPT representation for chromosome structure"
- [discussion] with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb): "with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb)"
- [methods] install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree')): "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [readme] a namesd list specifying the loci contained by all the clusters found by BHi-Cect: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
