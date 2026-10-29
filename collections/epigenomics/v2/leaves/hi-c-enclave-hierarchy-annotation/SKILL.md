---
name: hi-c-enclave-hierarchy-annotation
description: Use when a BHi-Cect output (nested tree list plus named per-cluster locus
  list) needs nestedness scoring and enrichment against epigenomic tracks to test
  whether nesting along the BPT relates to chromatin activity.
when_to_use_negative:
- When the goal is initial delineation of clusters from raw Hi-C contacts rather than
  annotating an existing BHi-Cect tree — use the BHi-Cect spectral clustering pipeline
  (Rao_matrix, pow_trans, spec_bipart) first
- When input Hi-C data lacks the resolution or enclave structure (no nested local-within-global
  clusters) needed to define top/bottom enclave anchors for NS scoring
- When no epigenomic annotation tracks for the matching cell type (e.g. IMR90) are
  available, since the enrichment contrast cannot be computed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BHi-Cect
  role: Produces the nested tree list and named per-cluster locus list that this skill
    annotates with NS scores and epigenomic enrichment
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Analysis environment; BHi-Cect functions are copy-pasted into the R session
- name: readr
  role: Loads the BHi-Cect output and Hi-C data files
- name: dplyr
  role: Data manipulation for counting and normalizing epigenomic overlap per enclave
- name: data.tree
  role: Parses the nested BPT hierarchy to identify top enclaves, bottom enclaves,
    and intermediate nodes for NS interpolation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:7fe91f7dfd552e2cfa5f933358ebfe038c95ae9600a4a0cc5da633ab733f7998
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hi-c-enclave-hierarchy-annotation

## Summary

Annotates the BHi-Cect partition tree (BPT) of Hi-C-derived chromosome enclaves with a nestedness (NS) score (0/0.5/1 anchoring of top enclaves, intermediate nodes, and bottom/leaf enclaves) and intersects enclave loci with epigenomic tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1) to test the top-vs-bottom enclave activity antagonism.

## When to use

Use when a BHi-Cect output (nested tree list plus named per-cluster locus list) needs nestedness scoring and enrichment against epigenomic tracks to test whether nesting along the BPT relates to chromatin activity.

## When NOT to use

- When the goal is initial delineation of clusters from raw Hi-C contacts rather than annotating an existing BHi-Cect tree — use the BHi-Cect spectral clustering pipeline (Rao_matrix, pow_trans, spec_bipart) first
- When input Hi-C data lacks the resolution or enclave structure (no nested local-within-global clusters) needed to define top/bottom enclave anchors for NS scoring
- When no epigenomic annotation tracks for the matching cell type (e.g. IMR90) are available, since the enrichment contrast cannot be computed

## Inputs

- BHi-Cect output: nested list defining the tree of the hierarchical structure plus a named list specifying the loci contained by all clusters
- 3-column sparse Hi-C contact matrix (e.g. IMR90 chr17/chr18/chr22)
- Epigenomic annotation tracks: DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1

## Outputs

- Per-cluster nestedness (NS) scores along the BPT
- Per-enclave overlap counts and normalized enrichment values for each epigenomic mark
- enclave_nestedness_enrichment.csv with NS score column plus enrichment columns for all listed marks
- Top- vs bottom-enclave enrichment contrast showing the activity antagonism

## How to apply

Load the BHi-Cect output — a nested list defining the tree and a named list of loci per cluster — in R with readr and dplyr. Parse the hierarchy with data.tree, classifying large global clusters as top enclaves and terminal local clusters as bottom enclaves, since enclaves are interwoven clusters at both local and global scales. Compute a nestedness (NS) score per cluster using top/bottom enclave and leaf anchors coded 0/0.5/1 with interpolation for intermediate nodes along the BPT. Intersect each enclave's loci with epigenomic annotation tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, and CTCF/RAD21/CHD1) counting overlaps per enclave, then normalize overlap counts per top enclave to obtain enrichment values per mark. Contrast top- versus bottom-enclave enrichment to reproduce the reported antagonism: progressive enrichment of epigenomic activity with nestedness in top enclaves versus progressive depletion in bottom enclaves, concentrating active regions at the enclave boundary. Write results to enclave_nestedness_enrichment.csv and validate the contrast qualitatively against the reported association between enclave nesting and epigenomic activity.

## Related tools

- **BHi-Cect** (Produces the nested tree list and named per-cluster locus list that this skill annotates with NS scores and epigenomic enrichment) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Analysis environment; BHi-Cect functions are copy-pasted into the R session)
- **readr** (Loads the BHi-Cect output and Hi-C data files)
- **dplyr** (Data manipulation for counting and normalizing epigenomic overlap per enclave)
- **data.tree** (Parses the nested BPT hierarchy to identify top enclaves, bottom enclaves, and intermediate nodes for NS interpolation)

## Examples

```
install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))
```

## Evaluation signals

- Output file enclave_nestedness_enrichment.csv exists and contains an NS score column plus enrichment columns for all listed marks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1)
- Top-versus-bottom contrast qualitatively matches the reported association: enrichment with nestedness in top enclaves, depletion in bottom enclaves
- NS anchors verify as 0 for top enclaves, 1 for bottom/leaf enclaves, 0.5 for leaf anchors, with intermediate nodes interpolated between anchors
- Every cluster in the named locus list from BHi-Cect receives an NS score and enrichment values (no missing clusters in output)
- GSEA on RNAP2 and H3K4me3 prevalence in bottom enclaves and CA of bottom-enclave TF content corroborate the enrichment trends

## Limitations

- Depending on Hi-C resolution, BHi-Cect highlights different scales: with 50 kb data it finds global clusters spanning whole chromosomes, with 5 kb data local clusters under 1 Mb — enclave definitions and NS scores are resolution-dependent
- Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from this analysis
- Cross-tissue variability of enclave arrangement is not examined and is flagged as future work
- Implementation as a stand-alone package is ongoing; BHi-Cect functions must be copy-pasted into an R session

## Evidence

- [abstract] We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA.: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [readme] output list structure: "a nested list defining the tree representing the hierarchical structure of the input matrix"
- [readme] hierarchical parsing tools: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [discussion] resolution dependence: "with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb)"
- [discussion] GSEA corroboration: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [abstract] enclave definition: "the hierarchical structure of chromosome is organized as 'enclaves', which are complex interwoven clusters at both local and global scales."
