---
name: multivariate-variance-explained-interpretation
description: Use when you have BHi-Cect's named list of loci per cluster (bottom enclaves)
  plus ChIP-seq peaks for a set of TFs and need to check whether a low-dimensional
  CA projection summarizes regulatory regimes across enclaves.
when_to_use_negative:
- Input is still raw 3-column sparse Hi-C contact data that has not been clustered
  by BHi-Cect — the CA operates on per-enclave TF counts, not on the contact matrix
  (build clusters first with Rao_matrix, pow_trans, spec_bipart).
- You have no TF binding data (e.g., only RNAP2/H3K4me3 tracks) and instead need GSEA-style
  prevalence enrichment in bottom enclaves — that is a different analysis in the article.
- The table is already a low-dimensional coordinate/embedding rather than raw counts
  per enclave — computing CA would double-transform the data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: BHi-Cect
  role: Produces the hierarchical clustering of Hi-C loci whose named cluster→loci
    list identifies the bottom enclaves used as rows/columns of the contingency table
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Environment in which the contingency table is assembled and correspondence
    analysis is run
- name: dplyr
  role: Used to count, for each bottom enclave, the number of loci bound by each TF
    and assemble the contingency table
- name: readr
  role: R package installed alongside the analysis stack; used for reading/writing
    data such as ca_results.csv
- name: caret
  role: R package installed as part of the BHi-Cect analysis stack cited for this
    task
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:2d59c7f076715063cb42a6fe50dddbe4bad04eddb14752d8236fff1a8d6c2049
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multivariate-variance-explained-interpretation

## Summary

Summarize and compare the regulatory contexts of BHi-Cect bottom (leaf) enclaves by running correspondence analysis (CA) on a TF-by-bottom-enclave contingency table of ChIP-seq binding counts, then interpret the leading CA axes via their explained variance (the first two dimensions capture ~63.3% of total inertia in the IMR90 analysis).

## When to use

Use when you have BHi-Cect's named list of loci per cluster (bottom enclaves) plus ChIP-seq peaks for a set of TFs and need to check whether a low-dimensional CA projection summarizes regulatory regimes across enclaves.

## When NOT to use

- Input is still raw 3-column sparse Hi-C contact data that has not been clustered by BHi-Cect — the CA operates on per-enclave TF counts, not on the contact matrix (build clusters first with Rao_matrix, pow_trans, spec_bipart).
- You have no TF binding data (e.g., only RNAP2/H3K4me3 tracks) and instead need GSEA-style prevalence enrichment in bottom enclaves — that is a different analysis in the article.
- The table is already a low-dimensional coordinate/embedding rather than raw counts per enclave — computing CA would double-transform the data.

## Inputs

- BHi-Cect output: nested list defining the tree plus a named list specifying the loci contained by all clusters (bottom/leaf enclaves identified from IMR90 Hi-C)
- ChIP-seq peak datasets for ten IMR90 TFs: CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1
- TF-by-bottom-enclave count contingency table built with dplyr in R

## Outputs

- ca_results.csv containing the TF-by-bottom-enclave contingency table, CA dimension coordinates, and variance summary
- Proportion of inertia explained by the first two CA dimensions (expected ~63.3%)
- Interpretation of the first CA axis as a gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime

## How to apply

Take BHi-Cect's output — a namesd list specifying the loci contained by all the clusters found — and identify the bottom (leaf) enclaves of the hierarchy. Load ChIP-seq peak data for the ten IMR90 transcription factors (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1) and use dplyr in R to count, for each bottom enclave, the number of loci bound by each TF; assemble these counts into a TF-by-bottom-enclave contingency table. Run correspondence analysis on this table in R and compute the proportion of inertia explained by the first two CA dimensions, verifying it reaches ~63.3%. Interpret the leading CA axis to confirm the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime. Save the contingency table together with the CA dimension coordinates and variance summary to ca_results.csv.

## Related tools

- **BHi-Cect** (Produces the hierarchical clustering of Hi-C loci whose named cluster→loci list identifies the bottom enclaves used as rows/columns of the contingency table) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Environment in which the contingency table is assembled and correspondence analysis is run)
- **dplyr** (Used to count, for each bottom enclave, the number of loci bound by each TF and assemble the contingency table)
- **readr** (R package installed alongside the analysis stack; used for reading/writing data such as ca_results.csv)
- **caret** (R package installed as part of the BHi-Cect analysis stack cited for this task)

## Examples

```
install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))  # then in R: ca_res <- ca(as.table(tf_by_enclave_table)); write_csv(ca_results, 'ca_results.csv')
```

## Evaluation signals

- The first two CA dimensions account for ~63.3% of the total inertia of the TF-by-bottom-enclave contingency table.
- The first CA axis reproduces the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime.
- The contingency table contains exactly the ten TFs named in the workflow (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1) scored against the BHi-Cect bottom enclaves.
- ca_results.csv contains three components: the contingency table, CA dimension coordinates, and the variance summary.
- Row/column counts of the contingency table match the number of bottom (leaf) enclaves reported by BHi-Cect for the IMR90 Hi-C data.

## Limitations

- The CA is descriptive: it summarizes TF-content variation across enclaves but does not establish regulatory mechanism, and the article notes details of competing molecular dynamics cannot be inferred with the present analysis.
- No software version is stated for BHi-Cect or the analysis software, so exact reproducibility of the 63.3% figure may vary with package versions.
- The analysis is anchored to IMR90 Hi-C data (chr17, chr22, chr18) and the ten named TFs; cross-tissue variability of enclave arrangement was not examined and is flagged as future work.

## Evidence

- [discussion] CA on TF-content table: "Our CA results highlight how the bottom enclave's TF-content shapes distinct regulatory regimes"
- [methods] R analysis stack: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
- [readme] BHi-Cect cluster output used as input: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [other] task finding on variance explained: [evidence span withheld — anchor status: not_found]
