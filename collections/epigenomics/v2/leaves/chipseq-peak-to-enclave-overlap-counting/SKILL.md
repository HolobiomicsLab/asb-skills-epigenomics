---
name: chipseq-peak-to-enclave-overlap-counting
description: Use when BHi-Cect cluster output (named lists of loci per enclave) is
  available and you need to characterize each bottom enclave's regulatory content
  by quantifying ChIP-seq TF binding per enclave.
when_to_use_negative:
- You need cluster identification itself rather than functional annotation of existing
  clusters — run the BHi-Cect spectral clustering pipeline (Rao_matrix, pow_trans,
  spec_bipart) first.
- The epigenomic question concerns enrichment signatures rather than per-enclave TF
  counts — the paper used GSEA on RNAP2 and H3K4me3 prevalence in bottom enclaves
  for that purpose.
- Input Hi-C data has not been converted from the 3-column sparse contact format;
  the contingency step presupposes a completed BHi-Cect tree of enclaves.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3175
tools:
- name: BHi-Cect
  role: Produces the nested hierarchy and the named list of loci per cluster from
    which bottom enclaves are identified
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Counts, for each bottom enclave, the number of loci bound by each transcription
    factor and assembles the contingency table
- name: R
  role: Host environment for BHi-Cect functions, dplyr counting, correspondence analysis,
    and saving results
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: readr
  role: Reads the 3-column sparse Hi-C matrix file and writes the contingency table
    / CA results (e.g. ca_results.csv)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:33a2cfcdfdeb4b0c2ce6c61720dab790ad1058c1047a71eb99bf7a1646654a28
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chipseq-peak-to-enclave-overlap-counting

## Summary

Counts, for each BHi-Cect bottom (leaf) enclave, the number of loci bound by each transcription factor from ChIP-seq peak data, assembling a TF-by-enclave contingency table that can be summarized by correspondence analysis (CA).

## When to use

Use when BHi-Cect cluster output (named lists of loci per enclave) is available and you need to characterize each bottom enclave's regulatory content by quantifying ChIP-seq TF binding per enclave.

## When NOT to use

- You need cluster identification itself rather than functional annotation of existing clusters — run the BHi-Cect spectral clustering pipeline (Rao_matrix, pow_trans, spec_bipart) first.
- The epigenomic question concerns enrichment signatures rather than per-enclave TF counts — the paper used GSEA on RNAP2 and H3K4me3 prevalence in bottom enclaves for that purpose.
- Input Hi-C data has not been converted from the 3-column sparse contact format; the contingency step presupposes a completed BHi-Cect tree of enclaves.

## Inputs

- BHi-Cect output: named list specifying the loci contained by all clusters found by BHi-Cect (leaf/bottom enclaves selected from it)
- ChIP-seq peak data (peak calls/loci) for the ten TFs in IMR90: CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1

## Outputs

- TF-by-bottom-enclave contingency table of binding locus counts per TF per enclave
- Correspondence analysis dimension coordinates and variance summary (e.g. ca_results.csv)

## How to apply

Run BHi-Cect on IMR90 Hi-C data using the README pipeline (read the 3-column sparse matrix with readr::read_delim, convert with Rao_matrix, power transform with pow_trans, build the graph with graph_from_adjacency_matrix, cluster with spec_bipart) and use the returned named list specifying the loci contained by all clusters to identify bottom (leaf) enclaves. Load ChIP-seq peak data for the ten TFs in IMR90 (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1) and use dplyr to count, for each bottom enclave, the number of loci bound by each TF. Assemble the counts into a TF-by-bottom-enclave contingency table, which downstream correspondence analysis uses to summarize the regulatory regimes of enclaves (e.g. the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regime). Save the table with CA coordinates and variance summary, e.g. to ca_results.csv. The decision point is choosing which clusters in the BHi-Cect tree are bottom enclaves: these are the leaves of the hierarchy, not the nested internal clusters.

## Related tools

- **BHi-Cect** (Produces the nested hierarchy and the named list of loci per cluster from which bottom enclaves are identified) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Counts, for each bottom enclave, the number of loci bound by each transcription factor and assembles the contingency table)
- **R** (Host environment for BHi-Cect functions, dplyr counting, correspondence analysis, and saving results) — https://github.com/princeps091-binf/BHi-Cect
- **readr** (Reads the 3-column sparse Hi-C matrix file and writes the contingency table / CA results (e.g. ca_results.csv))

## Examples

```
chr_dat <- read_delim('./path/to/hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res <- spec_bipart(chr_pow,g_chr1)
```

## Evaluation signals

- Correspondence analysis of the resulting TF-by-enclave contingency table shows the first two CA dimensions account for ~63.3% of total variance.
- The leading CA axis reproduces the gradient separating MAFK from the MAZ-MXI-RCOR1/CEBP-FOS regulatory regime.
- The contingency table has one column per bottom enclave (leaf of the BHi-Cect tree, not internal clusters) and rows covering the ten TFs (CEBP, RCOR1, MAZ, MXI, RFX5, MafK, FOS, USF2, NFE2L2, ELK1).
- Counts are non-negative integers and every counted locus is present in both the BHi-Cect cluster's named locus list and the corresponding TF ChIP-seq peak set.
- ca_results.csv contains the contingency table together with CA dimension coordinates and the variance summary.

## Limitations

- Downstream CA results depend on choosing the correct bottom (leaf) enclaves from the BHi-Cect tree; nesting of local clusters within global clusters means enclave membership is not genomic contiguity-based.
- Only ten TFs in IMR90 are covered by this analysis; other TFs or cell lines (e.g. K562) require new peak data, which the paper references without accession IDs.
- BHi-Cect is a function set to copy-paste into an R session ('Implementation as a stand-alone package ongoing'), so the pipeline is script-based rather than packaged.
- Mechanistic details of the regulatory regimes (competing molecular dynamics such as loop extrusion vs phase separation) cannot be inferred with the present analysis.

## Evidence

- [discussion] Correspondence analysis (CA) of bottom enclave TF-content: [evidence span withheld — anchor status: not_found]
- [readme] a namesd list specifying the loci contained by all the clusters found by BHi-Cect: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [other] ChIP-seq peak data for the ten transcription factors in IMR90: [evidence span withheld — anchor status: not_found]
- [readme] load the 3-column sparse matrix Hi-C data: "#load the 3-column sparse matrix Hi-C data"
- [discussion] enclaves are mostly non-contiguous and deeply interwoven clusters: "A key finding are enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
