---
name: epigenomic-feature-enrichment-analysis
description: Use when BHi-Cect enclave loci must be intersected with epigenomic tracks
  (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1) to test whether
  activity varies with nestedness along the BPT.
when_to_use_negative:
- The Hi-C contact matrix has not yet been clustered with BHi-Cect (Rao_matrix → pow_trans
  → spec_bipart) — no enclave locus lists exist to intersect with the tracks.
- Input is already a downstream result table (e.g. GSEA of RNAP2/H3K4me3 prevalence
  in bottom enclaves or CA of bottom-enclave TF content) rather than BHi-Cect clusters
  plus raw annotation tracks.
- The goal is mechanistic interpretation of chromatin architecture (loop extrusion
  vs phase separation), which the article states cannot be inferred from the present
  analysis.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3308
tools:
- name: BHi-Cect
  role: Produces the nested tree list and named per-cluster locus lists whose enclaves
    are NS-scored and intersected with epigenomic tracks
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: 'Runtime environment: BHi-Cect functions are copy-pasted into an R session
    and the enrichment workflow runs there'
- name: data.tree
  role: Parses the BPT hierarchy to classify top (large global) versus bottom (terminal
    local) enclaves and interpolate NS anchors coded 0/0.5/1
- name: dplyr
  role: Joins and aggregates per-enclave overlap counts and normalizes enrichment
    per top enclave before the top/bottom contrast
- name: readr
  role: Loads the 3-column sparse Hi-C / locus inputs and writes enclave_nestedness_enrichment.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:4a27f8c7fbcf59a176a33189536eb976f989d3a60cbb02b6e1091908abea2204
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# epigenomic-feature-enrichment-analysis

## Summary

Scores each BHi-Cect enclave's nestedness (NS in [0,1] via 0/0.5/1 anchors with interpolation along the BPT) and normalizes per-enclave overlap counts with DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2 and CTCF/RAD21/CHD1 tracks to quantify epigenomic activity along the BHi-Cect partition tree. Reproduces the reported antagonism: activity progressively enriched with NS in top enclaves, progressively depleted with NS in bottom enclaves.

## When to use

Use when BHi-Cect enclave loci must be intersected with epigenomic tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, CTCF/RAD21/CHD1) to test whether activity varies with nestedness along the BPT.

## When NOT to use

- The Hi-C contact matrix has not yet been clustered with BHi-Cect (Rao_matrix → pow_trans → spec_bipart) — no enclave locus lists exist to intersect with the tracks.
- Input is already a downstream result table (e.g. GSEA of RNAP2/H3K4me3 prevalence in bottom enclaves or CA of bottom-enclave TF content) rather than BHi-Cect clusters plus raw annotation tracks.
- The goal is mechanistic interpretation of chromatin architecture (loop extrusion vs phase separation), which the article states cannot be inferred from the present analysis.

## Inputs

- BHi-Cect output in R: a nested list defining the tree plus a named list specifying the loci contained by all the clusters found by BHi-Cect
- Epigenomic annotation tracks (interval/BED-style) for DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2 and CTCF/RAD21/CHD1
- 3-column sparse Hi-C contact matrix (e.g. IMR90 chr17/chr18/chr22) if the BPT must be regenerated via Rao_matrix, pow_trans and spec_bipart

## Outputs

- enclave_nestedness_enrichment.csv: per-enclave table with an NS score column plus enrichment columns for all listed marks
- Per-enclave locus-overlap counts against each epigenomic annotation track
- Top- versus bottom-enclave contrast of normalized enrichment showing the reported antagonism

## How to apply

Load the BHi-Cect output (nested tree list plus the named list of loci per cluster) in R using readr and dplyr, then parse the hierarchy with data.tree to flag top enclaves (large global clusters) and bottom enclaves (terminal local clusters). Compute the nestedness (NS) score per cluster by coding top-enclave, intermediate, and bottom-enclave/leaf anchors as 0/0.5/1 with interpolation for intermediate nodes, so NS encodes position along the enclave hierarchy. Intersect each enclave's loci with the epigenomic annotation tracks (DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2, and CTCF/RAD21/CHD1), counting overlaps per enclave. Normalize the overlap counts per top enclave to obtain a feature enrichment value for each mark. Contrast top- versus bottom-enclave enrichment to reproduce the reported epigenomic activity antagonism — progressive enrichment with NS within top enclaves versus progressive depletion within bottom enclaves, concentrating relatively more active regions at the top/bottom enclave boundary — and write the results to enclave_nestedness_enrichment.csv. The rationale is the paper's finding that the nesting of local clusters within global clusters characterizing enclaves is associated with the epigenomic activity found on the underlying DNA.

## Related tools

- **BHi-Cect** (Produces the nested tree list and named per-cluster locus lists whose enclaves are NS-scored and intersected with epigenomic tracks) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Runtime environment: BHi-Cect functions are copy-pasted into an R session and the enrichment workflow runs there)
- **data.tree** (Parses the BPT hierarchy to classify top (large global) versus bottom (terminal local) enclaves and interpolate NS anchors coded 0/0.5/1)
- **dplyr** (Joins and aggregates per-enclave overlap counts and normalizes enrichment per top enclave before the top/bottom contrast)
- **readr** (Loads the 3-column sparse Hi-C / locus inputs and writes enclave_nestedness_enrichment.csv)

## Examples

```
library(dplyr); library(data.tree); write_csv(enclave_nestedness_enrichment, 'enclave_nestedness_enrichment.csv')
```

## Evaluation signals

- enclave_nestedness_enrichment.csv exists and contains an NS score column plus enrichment columns for DNase, LAD, ChromHMM, CAGE, RNA-seq, TF, RNAP2 and CTCF/RAD21/CHD1
- NS scores are bounded in [0,1] with anchors coded 0 (top enclave), 0.5 (intermediate) and 1 (bottom enclave/leaf)
- Within top enclaves enrichment is progressively higher and within bottom enclaves progressively lower at increasing NS — the qualitative antagonism matches the reported association
- Normalized per-mark enrichment values are non-negative, finite, and reproducible from the raw per-enclave overlap counts
- Directional check against the abstract: nesting of local clusters within global clusters is associated with epigenomic activity of the underlying DNA

## Limitations

- Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from this enrichment analysis
- Cross-tissue variability of enclave arrangement is unexamined; the article flags examining how the arrangement of enclaves varies across tissues as future work
- BHi-Cect's cluster scale depends on Hi-C resolution (50 kb data highlights global chromosome clusters; 5 kb data focuses on local clusters under 1 Mb), which changes the enclave set and NS axis being scored
- Cell lines (IMR90, K562) are referenced without dataset accession IDs, so provenance of the epigenomic tracks must be supplied externally

## Evidence

- [abstract] We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA.: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [discussion] GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [discussion] Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [discussion] Our results also suggest that depending on the position along the enclave hierarchy the cluster is found: "Our results also suggest that depending on the position along the enclave hierarchy the cluster is found"
- [discussion] Mechanistic details of competing molecular dynamics (loop extrusion vs phase separation) cannot be inferred from the present analysis: [evidence span withheld — anchor status: not_found]
- [methods] install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree')): "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [readme] a namesd list specifying the loci contained by all the clusters found by BHi-Cect: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [readme] a nested list defining the tree representing the hierarchical structure of the input matrix: "a nested list defining the tree representing the hierarchical structure of the input matrix"
