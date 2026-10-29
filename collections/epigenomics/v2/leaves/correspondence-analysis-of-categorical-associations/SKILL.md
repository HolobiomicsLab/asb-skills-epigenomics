---
name: correspondence-analysis-of-categorical-associations
description: Use when you have a two-way contingency table of categorical associations
  (e.g. TF binding site content per BHi-Cect bottom enclave) and want to summarize
  rows and columns as gradients on CA dimensions.
when_to_use_negative:
- When the data are continuous measurements rather than counts over a two-way categorical
  contingency table (e.g. raw ChIP signal intensities per locus).
- When the goal is testing differential TF binding significance between enclaves rather
  than summarizing categorical associations as low-dimensional axes (a set-enrichment/GSEA-style
  approach is used elsewhere in the paper for RNAP2 and H3K4me3 prevalence).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0203
tools:
- name: BHi-Cect
  role: Produces the hierarchical clustering of IMR90 Hi-C loci; its named list of
    loci per cluster identifies the bottom enclaves that define the contingency-table
    columns
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Environment in which the CA is run and the BHi-Cect functions are executed
- name: dplyr
  role: Used to count, for each bottom enclave, the number of loci bound by each TF
    when assembling the contingency table
- name: readr
  role: Used to read/write tabular data (contingency table and CA results saved via
    ca_results.csv)
- name: caret
  role: Listed among the R packages installed for the BHi-Cect analysis workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:8df92350b08b4ab65bacd81ed0ff8aeeed61cb5a3c045153c34cc492cee2c1d7
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# correspondence-analysis-of-categorical-associations

## Summary

Correspondence analysis (CA) applied to a TF-by-bottom-enclave contingency table derived from BHi-Cect's named cluster lists and IMR90 ChIP-seq peak data, to summarize the regulatory regimes of chromosome enclaves in a few orthogonal dimensions.

## When to use

Use when you have a two-way contingency table of categorical associations (e.g. TF binding site content per BHi-Cect bottom enclave) and want to summarize rows and columns as gradients on leading CA dimensions.

## When NOT to use

- When the data are continuous measurements rather than counts over a two-way categorical contingency table (e.g. raw ChIP signal intensities per locus).
- When the goal is testing differential TF binding significance between enclaves rather than summarizing categorical associations as low-dimensional axes (a set-enrichment/GSEA-style approach is used elsewhere in the paper for RNAP2 and H3K4me3 prevalence).

## Inputs

- BHi-Cect output for IMR90 Hi-C data (named list specifying the loci contained by each cluster, used to identify bottom/leaf enclaves)
- ChIP-seq peak data for ten IMR90 TFs (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1)

## Outputs

- TF-by-bottom-enclave contingency table of binding site counts
- CA dimension coordinates for TFs and enclaves
- Variance (inertia) summary per CA dimension (~63.3% on first two dimensions)
- ca_results.csv containing the contingency table, CA coordinates and variance summary

## How to apply

First build the contingency table: load the BHi-Cect output for IMR90 Hi-C data, use its named list of loci per cluster to identify the bottom (leaf) enclaves, then load ChIP-seq peak data for the ten IMR90 TFs (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1) and use dplyr in R to count, per bottom enclave, the number of loci bound by each TF. Assemble the counts into a TF-by-bottom-enclave contingency table and run correspondence analysis on it in R. Compute the proportion of inertia explained by the first two CA dimensions — in this paper's analysis it reaches ~63.3%, confirming the TF-content variety across bottom enclaves is well summarized by two dimensions. Interpret the leading axis, which should recover the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime. Save the contingency table together with the CA dimension coordinates and variance summary to ca_results.csv.

## Related tools

- **BHi-Cect** (Produces the hierarchical clustering of IMR90 Hi-C loci; its named list of loci per cluster identifies the bottom enclaves that define the contingency-table columns) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Environment in which the CA is run and the BHi-Cect functions are executed)
- **dplyr** (Used to count, for each bottom enclave, the number of loci bound by each TF when assembling the contingency table)
- **readr** (Used to read/write tabular data (contingency table and CA results saved via ca_results.csv))
- **caret** (Listed among the R packages installed for the BHi-Cect analysis workflow)

## Examples

```
chr_dat <- read_delim('path/to/hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); chr_spec_res <- spec_bipart(chr_pow, g_chr1)
```

## Evaluation signals

- The first two CA dimensions capture ~63.3% of the total variance (inertia) of the TF-content contingency table.
- The leading CA axis reproduces the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime.
- The contingency table row/column counts match the number of TFs (10) and bottom enclaves identified by BHi-Cect.
- ca_results.csv contains the contingency table, CA dimension coordinates and variance summary.

## Limitations

- CA depends on the prior BHi-Cect clustering; the paper notes the analysis cannot infer mechanistic details such as loop extrusion vs phase separation dynamics underlying the observed regimes.
- The README notes the algorithm implementation 'as a stand-alone package ongoing', so functions must be copy-pasted into the R session, which may affect reproducibility.
- No software versions are stated for BHi-Cect or the analysis software.

## Evidence

- [discussion] CA summarizes TF content of bottom enclaves: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [readme] BHi-Cect output is a named list of loci per cluster: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [methods] R packages required for the workflow: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [discussion] Enclaves are novel mostly non-contiguous clusters: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [supplementary] Source code repository: "The code for the algorithm is available at https://github.com/princeps091-binf/BHi-Cect."
