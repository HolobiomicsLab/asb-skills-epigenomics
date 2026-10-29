---
name: hic-cluster-loci-annotation
description: Use when BHi-Cect spectral clustering of IMR90 (or similar) Hi-C data
  has produced a tree of enclaves plus a named list of loci per cluster, and you need
  to relate enclave regulatory content to ChIP-seq data (e.g. ten TFs) via contingency-table
  analysis.
when_to_use_negative:
- Your goal is TAD-style segmentation that assumes clusters of loci contiguous along
  the genome coordinate — BHi-Cect clustering and its enclave annotation explicitly
  avoid genomic-contiguity assumptions.
- The clusters already have epigenomic counts assembled (input is already a TF-by-enclave
  feature/contingency table) — proceed directly to CA rather than re-counting peaks.
- No matching ChIP-seq or other epigenomic annotation is available for the cell line
  used to generate the Hi-C data — the clustering output alone gives the tree and
  locus lists, not TF-content summaries.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BHi-Cect
  role: Top-down spectral clustering of Hi-C data producing the tree of enclaves and
    the named list of loci per cluster that defines the bottom enclaves to annotate
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Counting, per bottom enclave, the number of loci bound by each transcription
    factor when building the contingency table
- name: readr
  role: Loading the 3-column sparse Hi-C matrix file and writing the contingency table
    / CA results (ca_results.csv)
- name: caret
  role: One of the R packages required by the BHi-Cect script for the clustering/annotation
    workflow
- name: R
  role: Environment in which BHi-Cect functions are sourced and correspondence analysis
    of the TF-content table is run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:312383fb3832bfc4f5324019875d2001b5ed2519ab118d89f6cdd8d0ca0be28d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - caret
  - dplyr
  - readr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hic-cluster-loci-annotation

## Summary

Annotate BHi-Cect chromosome clusters (bottom/leaf enclaves) with epigenomic content by counting TF ChIP-seq binding per enclave and summarizing regulatory regimes via correspondence analysis on the TF-by-enclave contingency table.

## When to use

Use when BHi-Cect spectral clustering of IMR90 (or similar) Hi-C data has produced a tree of enclaves plus a named list of loci per cluster, and you need to relate enclave regulatory content to ChIP-seq data (e.g. ten TFs) via contingency-table analysis.

## When NOT to use

- Your goal is TAD-style segmentation that assumes clusters of loci contiguous along the genome coordinate — BHi-Cect clustering and its enclave annotation explicitly avoid genomic-contiguity assumptions.
- The clusters already have epigenomic counts assembled (input is already a TF-by-enclave feature/contingency table) — proceed directly to CA rather than re-counting peaks.
- No matching ChIP-seq or other epigenomic annotation is available for the cell line used to generate the Hi-C data — the clustering output alone gives the tree and locus lists, not TF-content summaries.

## Inputs

- BHi-Cect output list: nested list defining the tree plus a named list of loci contained by all clusters found
- 3-column sparse Hi-C contact matrix file (e.g. IMR90 Hi-C data)
- ChIP-seq peak data for transcription factors in the matching cell line (e.g. ten TFs in IMR90: CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1)

## Outputs

- TF-by-bottom-enclave contingency count table
- CA dimension coordinates and variance summary (e.g. saved to ca_results.csv)
- Inertia/variance proportions for the first two CA dimensions (~63.3% expected in the reference analysis)

## How to apply

Load the BHi-Cect output for the Hi-C dataset (e.g. IMR90), which returns a nested tree list and a named list specifying the loci contained by all clusters; select the bottom (leaf) enclaves from the tree. Load ChIP-seq peak data for the transcription factors (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1) and use dplyr to count, per bottom enclave, how many of its loci are bound by each TF. Assemble these counts into a TF-by-bottom-enclave contingency table and run correspondence analysis (CA) on it in R. Check the proportion of inertia explained by the first two dimensions (the article's IMR90 analysis reached ~63.3% of total variance) and interpret the leading CA axis for regulatory regimes such as the MAFK versus MAZ-MXI-RCOR1/CEBP-FOS gradient.

## Related tools

- **BHi-Cect** (Top-down spectral clustering of Hi-C data producing the tree of enclaves and the named list of loci per cluster that defines the bottom enclaves to annotate) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Counting, per bottom enclave, the number of loci bound by each transcription factor when building the contingency table)
- **readr** (Loading the 3-column sparse Hi-C matrix file and writing the contingency table / CA results (ca_results.csv))
- **caret** (One of the R packages required by the BHi-Cect script for the clustering/annotation workflow)
- **R** (Environment in which BHi-Cect functions are sourced and correspondence analysis of the TF-content table is run)

## Examples

```
chr_dat <- read_delim('./path/to/hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res <- spec_bipart(chr_pow,g_chr1)
```

## Evaluation signals

- The first two CA dimensions account for ~63.3% of the total variance of the TF-content contingency table.
- The first CA axis reproduces the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime.
- The number of bottom enclaves and their locus membership match the named list of loci in the BHi-Cect output (leaves of the nested tree).
- Contingency table row/column margins reconcile with the total loci per enclave and total peaks per TF.
- ca_results.csv contains the contingency table together with CA dimension coordinates and the variance summary.

## Limitations

- Nesting of local clusters within global enclaves is associated with epigenomic activity, but mechanistic details (e.g. loop extrusion vs phase separation) cannot be inferred with the present analysis.
- No software versions are stated for BHi-Cect or the analysis software, so exact reproducibility of versions is uncontrolled.
- Implementation as a stand-alone package is ongoing; users must copy-paste the BHi-Cect functions into an R session.

## Evidence

- [readme] BHi-Cect output provides a nested list defining the tree and a named list of loci per cluster, which is the substrate for annotation.: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [discussion] Correspondence analysis of bottom enclave TF-content is an article workflow step: [evidence span withheld — anchor status: not_found]
- [discussion] The CA results interpret bottom enclave TF-content as distinct regulatory regimes.: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [methods] BHi-Cect is run in R by copying its functions into the session.: "Copy-paste the different functions present in the BHi-Cect script within your R session"
- [abstract] Enclaves are interwoven clusters whose nesting is linked to epigenomic activity of the underlying DNA.: "the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
