---
name: resnik-semantic-similarity-of-go-terms
description: Use when chromosome clusters (e.g. BHi-Cect bottom enclaves from IMR90
  Hi-C data) need an ontology-based functional similarity score to correlate against
  BPT distance or genomic coordinate distance.
when_to_use_negative:
- When the functional similarity measure should be TF-content distance in CA space
  — that is computed from TF-content tables, not from enriched GO-terms
- When bottom enclaves lack enriched GO-terms (e.g. clusters with too few annotated
  loci), since Resnik similarity cannot be computed between empty enrichment sets
- When the question is purely structural clustering of Hi-C loci — use BHi-Cect's
  spectral clustering (spec_bipart) rather than functional similarity scoring
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: BHi-Cect
  role: Produces the nested tree list and named list of loci per cluster (spec_bipart)
    whose bottom enclaves are the units being compared by semantic similarity
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Analysis environment in which GO enrichment, Resnik similarity, correlation
    and plotting are implemented
- name: readr
  role: Used to read input data and write pairwise values (fig4c_bpt_functional_similarity.csv)
- name: data.tree
  role: Represents the BPT tree from which pairwise BPT distances between bottom enclaves
    are computed for the correlation
- name: dplyr
  role: Data manipulation of per-enclave enrichment results and pairwise value tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:ae291592166be7bbb5dbda49bc1f71ea419fce7c3e4152876d7195df63029ef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - RSpectra
  - caret
  - data.tree
  - dplyr
  - igraph
  - readr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# resnik-semantic-similarity-of-go-terms

## Summary

Quantifies functional similarity between bottom enclaves by computing Resnik semantic similarity between their enriched GO-terms, serving as an ontology-based measure of whether enclaves further apart in the BPT are functionally less alike. It complements the TF-content correspondence-analysis distance used to reproduce the Figure 4C trend.

## When to use

Use when chromosome clusters (e.g. BHi-Cect bottom enclaves from IMR90 Hi-C data) need an ontology-based functional similarity score to correlate against BPT distance or genomic coordinate distance.

## When NOT to use

- When the functional similarity measure should be TF-content distance in CA space — that is computed from TF-content tables, not from enriched GO-terms
- When bottom enclaves lack enriched GO-terms (e.g. clusters with too few annotated loci), since Resnik similarity cannot be computed between empty enrichment sets
- When the question is purely structural clustering of Hi-C loci — use BHi-Cect's spectral clustering (spec_bipart) rather than functional similarity scoring

## Inputs

- Named list of loci per cluster (second element of spec_bipart output) identifying bottom enclaves for the IMR90 chromosome
- Nested list defining the BPT tree of clusters (first element of spec_bipart output)
- Enriched GO-terms per bottom enclave from GO enrichment analysis

## Outputs

- Pairwise Resnik semantic similarity matrix between bottom enclaves
- Pairwise values written to fig4c_bpt_functional_similarity.csv (BPT distance vs functional similarity measures)
- Plot of BPT distance vs functional similarity showing the decreasing similarity trend

## How to apply

First run GO enrichment independently on the set of loci belonging to each bottom enclave, obtained from the named list of loci per cluster returned by spec_bipart. For each pair of bottom enclaves, compute Resnik semantic similarity between their enriched GO-terms, yielding a pairwise similarity matrix. Treat this semantic similarity as the second functional similarity measure alongside the euclidean distance in CA space of TF content. Correlate it against pairwise BPT distances (computed from the data.tree representation of the nested tree list) and against pairwise genomic coordinate distances as a control; the expected result is that Resnik similarity decreases as bottom enclaves are further apart in the BPT, reproducing the positive functional-similarity trend of Figure 4C, and is more strongly related to BPT distance than to coordinate distance. Write all pairwise values to fig4c_bpt_functional_similarity.csv.

## Related tools

- **BHi-Cect** (Produces the nested tree list and named list of loci per cluster (spec_bipart) whose bottom enclaves are the units being compared by semantic similarity) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Analysis environment in which GO enrichment, Resnik similarity, correlation and plotting are implemented)
- **readr** (Used to read input data and write pairwise values (fig4c_bpt_functional_similarity.csv))
- **data.tree** (Represents the BPT tree from which pairwise BPT distances between bottom enclaves are computed for the correlation)
- **dplyr** (Data manipulation of per-enclave enrichment results and pairwise value tables)

## Evaluation signals

- The correlation between BPT distance and Resnik semantic similarity must be negative (similarity decreases as bottom enclaves are further apart in the BPT), reproducing the positive functional-similarity trend of Figure 4C
- The BPT-distance vs functional-similarity relation must be clearly stronger than the correlation with the pairwise genomic coordinate-distance control
- The pairwise Resnik similarity matrix must be complete and symmetric over all bottom-enclave pairs
- All pairwise values (BPT distance, TF-content CA distance, Resnik similarity, coordinate distance) must be written to fig4c_bpt_functional_similarity.csv

## Limitations

- The relation is validated on IMR90 Hi-C data (chr17, chr22, chr18); cross-tissue variability of enclave arrangement was not examined and flagged as future work
- The BPT-distance vs functional-similarity relation integrates enclave function but mechanistic details of the underlying molecular dynamics cannot be inferred
- No software version is stated for BHi-Cect or any analysis software, so exact reproduction of Resnik similarity values may depend on tool versions

## Evidence

- [abstract] Furthermore, we show that the hierarchical nesting that links different enclaves integrates their respective function.: "Furthermore, we show that the hierarchical nesting that links different enclaves integrates their respective function."
- [discussion] CA of TF content as the complementary functional similarity measure: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [discussion] GSEA on bottom enclaves: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [methods] R package environment for the analysis: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
