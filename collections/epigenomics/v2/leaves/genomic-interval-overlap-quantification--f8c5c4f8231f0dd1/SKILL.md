---
name: genomic-interval-overlap-quantification
description: Use when you have BHi-Cect cluster/loci assignments and coordinate-based
  epigenomic annotation tracks, and need per-enclave overlap counts and enrichment
  values to test how activity varies with nestedness along the BPT.
when_to_use_negative:
- Input is still a raw 3-column sparse Hi-C contact matrix that has not yet been clustered
  by BHi-Cect — run the clustering pipeline (Rao_matrix, pow_trans, spec_bipart) first
- The goal is 3D structural characterization of clusters themselves (cluster maps,
  tree visualization) rather than functional/epigenomic annotation of cluster loci
- Annotations and Hi-C loci come from different genome builds or chromosomes, so interval
  overlap would be meaningless without prior liftOver/reconciliation
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: R
  role: Environment in which the BHi-Cect functions and the overlap/enrichment computations
    are executed
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Data manipulation for intersecting enclave loci with annotation tracks and
    normalizing overlap counts
- name: data.tree
  role: Parses the BPT hierarchy to identify top enclaves (large global clusters)
    and bottom enclaves (terminal local clusters)
- name: readr
  role: Loads the BHi-Cect output and annotation track files
- name: BHi-Cect
  role: Produces the nested tree and named per-cluster loci lists that define the
    intervals to be overlapped
  repo: https://github.com/princeps091-binf/BHi-Cect
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:b5366fba8a01e546f4b352747053a46dac3f978ef8f89aa743335e4b25f523d1
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-interval-overlap-quantification

## Summary

Quantifies epigenomic annotation coverage of chromosome clusters by intersecting BHi-Cect enclave loci with tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1) and normalizing overlap counts per enclave. This exposes the antagonistic enrichment/depletion of epigenomic activity along the BHi-Cect partition tree (BPT) nestedness axis.

## When to use

Use when you have BHi-Cect cluster/loci assignments and coordinate-based epigenomic annotation tracks, and need per-enclave overlap counts and enrichment values to test how activity varies with nestedness along the BPT.

## When NOT to use

- Input is still a raw 3-column sparse Hi-C contact matrix that has not yet been clustered by BHi-Cect — run the clustering pipeline (Rao_matrix, pow_trans, spec_bipart) first
- The goal is 3D structural characterization of clusters themselves (cluster maps, tree visualization) rather than functional/epigenomic annotation of cluster loci
- Annotations and Hi-C loci come from different genome builds or chromosomes, so interval overlap would be meaningless without prior liftOver/reconciliation

## Inputs

- BHi-Cect output: nested list defining the BPT tree plus named list of loci per cluster (R)
- Epigenomic annotation tracks for the same chromosome: DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1 (loci/intervals)
- Hi-C chromosome of interest (e.g., IMR90 chr17, chr18, chr22) matching the track genome build

## Outputs

- enclave_nestedness_enrichment.csv: one row per enclave with NS score column and enrichment columns for all listed epigenomic marks
- Top- versus bottom-enclave enrichment contrast showing the reported antagonism

## How to apply

Load the BHi-Cect output in R (a nested list defining the BPT plus a named list of loci per cluster) with readr and dplyr, and parse the hierarchy with data.tree to identify top enclaves (large global clusters) and bottom enclaves (terminal local clusters). Compute the nestedness (NS) score per cluster using top/bottom enclave and leaf anchors coded as 0/0.5/1, interpolating for intermediate nodes. For each enclave, count overlaps of its loci with each epigenomic annotation track (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, and CTCF/RAD21/CHD1). Normalize overlap counts per top enclave to obtain feature enrichment values for each mark, then contrast top- versus bottom-enclave enrichment: within top enclaves activity is progressively enriched with increasing nestedness, whereas within bottom enclaves it is progressively depleted, producing an antagonism that concentrates relatively more active regions at the top/bottom enclave boundary. Write results to enclave_nestedness_enrichment.csv.

## Related tools

- **R** (Environment in which the BHi-Cect functions and the overlap/enrichment computations are executed) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Data manipulation for intersecting enclave loci with annotation tracks and normalizing overlap counts)
- **data.tree** (Parses the BPT hierarchy to identify top enclaves (large global clusters) and bottom enclaves (terminal local clusters))
- **readr** (Loads the BHi-Cect output and annotation track files)
- **BHi-Cect** (Produces the nested tree and named per-cluster loci lists that define the intervals to be overlapped) — https://github.com/princeps091-binf/BHi-Cect

## Examples

```
enrichment <- sapply(enclave_loci, function(l) length(intersect(l, track_loci)) / length(l))
```

## Evaluation signals

- enclave_nestedness_enrichment.csv exists and contains an NS score column plus enrichment columns for all listed marks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1)
- Per-enclave overlap counts are non-negative integers and per-mark normalized enrichment values are bounded by the number of enclave loci
- Top-versus-bottom contrast qualitatively matches the reported association: enrichment with increasing nestedness in top enclaves, depletion in bottom enclaves
- NS scores fall in [0,1] consistent with the 0/0.5/1 anchor coding with interpolation
- Every cluster in the BHi-Cect loci list appears as a row in the output (no enclaves dropped during intersection)

## Limitations

- The reported association links enclave nesting to epigenomic activity but mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from this analysis
- Annotation tracks are chromosome- and cell-line-specific (e.g., IMR90 vs K562); cross-tissue variability of enclave arrangement was not examined and is flagged as future work
- BHi-Cect resolution depends on Hi-C input resolution: with low resolution data (50 kb) global clusters dominate, while with high resolution data (5 kb) local clusters under 1 Mb are recovered, affecting which enclaves are scored

## Evidence

- [abstract] Nesting of local clusters within global clusters characterizing enclaves is associated with epigenomic activity of the underlying DNA: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [readme] BHi-Cect output loci list: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [readme] R package stack for the analysis: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [discussion] GSEA of epigenomic marks in bottom enclaves: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [discussion] TF-content correspondence analysis: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
