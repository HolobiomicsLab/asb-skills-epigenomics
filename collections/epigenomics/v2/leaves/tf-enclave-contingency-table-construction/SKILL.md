---
name: tf-enclave-contingency-table-construction
description: Use when you have BHi-Cect cluster output (named list of loci per cluster)
  plus TF ChIP-seq peaks for the same cell line and need a count matrix summarizing
  TF-binding content per bottom enclave.
when_to_use_negative:
- Your cluster output is not from BHi-Cect or lacks a named list of loci per cluster
  — there is no enclave hierarchy to intersect with the peaks.
- You need genome-wide regulatory summaries at the top-level domain scale rather than
  the leaf (bottom enclave) level — the CA finding applies specifically to bottom
  enclaves.
- TF ChIP-seq data for the same cell line as the Hi-C data is unavailable — cross-cell-line
  peaks would not reflect the epigenomic activity underlying those enclaves.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0602
tools:
- name: BHi-Cect
  role: Produces the nested hierarchy and named list of loci per cluster from which
    bottom enclaves are identified
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Used to count, for each bottom enclave, the number of loci bound by each TF
    when assembling the contingency table
- name: readr
  role: Used to read input data and write the contingency table and CA results to
    ca_results.csv
- name: caret
  role: Listed among required R packages for the BHi-Cect analysis environment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:b98ecfad52ff8507daa8c45aad09d9fe8302811707f77c60b5d13b478e4b4f60
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tf-enclave-contingency-table-construction

## Summary

Constructs a transcription-factor (TF) by bottom-enclave contingency table by intersecting BHi-Cect leaf-cluster locus membership (from IMR90 Hi-C) with ChIP-seq peak data for ten TFs (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1), enabling downstream correspondence analysis of regulatory regimes.

## When to use

Use when you have BHi-Cect cluster output (named list of loci per cluster) plus TF ChIP-seq peaks for the same cell line and need a count matrix summarizing TF-binding content per bottom enclave.

## When NOT to use

- Your cluster output is not from BHi-Cect or lacks a named list of loci per cluster — there is no enclave hierarchy to intersect with the peaks.
- You need genome-wide regulatory summaries at the top-level domain scale rather than the leaf (bottom enclave) level — the CA finding applies specifically to bottom enclaves.
- TF ChIP-seq data for the same cell line as the Hi-C data is unavailable — cross-cell-line peaks would not reflect the epigenomic activity underlying those enclaves.

## Inputs

- BHi-Cect output: named list specifying the loci contained in each cluster (bottom/leaf enclaves identified from the hierarchy)
- IMR90 Hi-C data as a 3-column sparse contact matrix (chr17, chr22, chr18 used in the paper)
- ChIP-seq peak data for ten transcription factors in IMR90: CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1

## Outputs

- TF-by-bottom-enclave contingency table of bound-locus counts per enclave per TF
- ca_results.csv containing the contingency table, CA dimension coordinates, and variance summary

## How to apply

Run BHi-Cect on the 3-column sparse Hi-C contact matrix (Rao_matrix -> pow_trans -> graph_from_adjacency_matrix -> spec_bipart) and extract, from its named list of clusters, the bottom (leaf) enclaves of the hierarchy. Load TF ChIP-seq peak intervals for the ten TFs in the same cell line (IMR90), and use dplyr joins/counts to tally, for each bottom enclave, how many of its loci overlap a peak for each TF. Assemble these counts into a TF-by-bottom-enclave contingency table (rows or columns = TFs, cells = number of bound loci per enclave). This table is the direct input to correspondence analysis (CA), whose first two dimensions are expected to summarize the enclave TF-content structure. Save the contingency table (e.g. with readr) alongside the CA coordinates and variance summary for validation.

## Related tools

- **BHi-Cect** (Produces the nested hierarchy and named list of loci per cluster from which bottom enclaves are identified) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Used to count, for each bottom enclave, the number of loci bound by each TF when assembling the contingency table)
- **readr** (Used to read input data and write the contingency table and CA results to ca_results.csv)
- **caret** (Listed among required R packages for the BHi-Cect analysis environment)

## Examples

```
chr_dat <- read_delim('./path/to/hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow, weighted=T); chr_spec_res <- spec_bipart(chr_pow, g_chr1)
```

## Evaluation signals

- The contingency table dimensions match the number of bottom enclaves by the ten TFs (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1), with non-negative integer counts.
- Row/column sums of the table are consistent with the total number of loci in each bottom enclave from the BHi-Cect named list.
- Downstream correspondence analysis on the table yields first two CA dimensions explaining ~63.3% of total variance.
- The leading CA axis reproduces the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime.
- ca_results.csv contains the contingency table together with CA dimension coordinates and a variance summary.

## Limitations

- Validation of the CA structure (~63.3% variance on the first two dimensions; MAFK vs MAZ-MXI-RCOR1/CEBP-FOS gradient) is specific to the IMR90 dataset and the ten TFs used; other TF panels or cell lines are unvalidated.
- No software versions are stated for BHi-Cect or the analysis stack, so exact reproducibility may vary.
- The intersection of ChIP-seq peaks with enclave loci depends on the resolution of the Hi-C data (e.g. 5 kb vs 50 kb), which changes what counts as a locus.

## Evidence

- [discussion] CA of bottom enclave TF-content: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [readme] Output format of BHi-Cect: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [readme] Required R packages: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [abstract] Enclaves as interwoven clusters: "the hierarchical structure of chromosome is organized as 'enclaves', which are complex interwoven clusters at both local and global scales"
- [readme] Input format expectation: "Expects a 3-column sparse contact matrix as input"
